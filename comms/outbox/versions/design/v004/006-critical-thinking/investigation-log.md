# Investigation Log — v004 Task 006

## I1: R1 — Scan API External Consumers

**Query**: Searched for all files referencing `videos/scan` beyond tests and design docs.

**Files found**: `src/stoat_ferret/api/routers/videos.py` (endpoint definition), `src/stoat_ferret/api/services/scan.py` (service), `src/stoat_ferret/api/schemas/video.py` (schemas), `tests/test_api/test_videos.py` (tests), plus design docs.

**Finding**: No external consumers exist. The scan endpoint is consumed only by the test suite. The GUI (v005) is not yet implemented. The API has no CLI client, no webhook consumers, and no external integrations.

**Conclusion**: R1 severity reduced from High to Low. Breaking change is safe — only test migration required.

## I2: R2 — Deepcopy Performance on Domain Objects

**Query**: Read `src/stoat_ferret/db/models.py` and `src/stoat_ferret/db/project_repository.py`.

**Finding**: `Project` is a simple dataclass with 7 scalar fields (str, int, datetime). `Video` has 14 scalar fields. No nested objects, no lists, no references to other domain objects. `AsyncInMemoryProjectRepository` stores `dict[str, Project]` — flat dictionary of flat dataclasses.

**Conclusion**: R2 severity reduced from Medium to Low. `copy.deepcopy()` on flat dataclasses with only scalar fields is trivially fast (microseconds). No performance concern.

## I3: R3 — FTS5 vs InMemory Search Semantics

**Query**: Read both search implementations.

**SQLite FTS5** (async_repository.py:186-198): Uses `MATCH` with `'"{query}"*'` — quoted phrase with trailing `*`. This means: treat the query as a literal phrase, then prefix-match. "test" matches "testing" but "est" does NOT match "testing".

**InMemory** (async_repository.py:301-309 and repository.py:329-337): Uses `query_lower in v.filename.lower()` — substring match. "est" DOES match "testing". Also matches against both filename and path fields.

**Gap analysis**:
1. **Substring vs prefix**: InMemory matches "est" in "testing"; FTS5 does not. This is the primary discrepancy.
2. **Phrase matching**: FTS5 treats multi-word queries as phrases; InMemory doesn't distinguish.
3. **Field scope**: InMemory searches filename+path; FTS5 searches whatever is indexed in `videos_fts` (needs FTS5 table definition check).
4. **Case sensitivity**: Both are case-insensitive (FTS5 default, InMemory uses `.lower()`).

**FTS5 behaviors NOT replicable in pure Python**: Full FTS5 syntax (AND, OR, NOT, NEAR), ranking by relevance, column filters. None of these are used by the application — only simple quoted phrase prefix search.

**Conclusion**: R3 is well-characterized. Per-token `startswith` matching will close the primary gap. Remaining differences (multi-word phrase handling) are minor and can be documented as known limitations.

## I4: R4 — validate_path Security Gap Assessment

**Query**: Read `rust/stoat_ferret_core/src/sanitize/mod.rs` (validate_path at line 230-238) and `src/stoat_ferret/api/services/scan.py`.

**Rust validate_path**: Only checks for empty string and null bytes. No path traversal (`../`), symlink, or directory allowlist checks. Comment at line 206: "Full directory allowlist validation... is deferred to the Python layer."

**Python path handling** (scan.py): `scan_directory()` receives a user-provided path, converts to `Path(path)`, checks `root.is_dir()`, then globs within it. Uses `file_path.absolute()` to resolve. No explicit traversal check, no symlink check, no directory allowlist.

**Grep for path traversal/symlink checks**: Only 2 hits — both are database path resolution in settings.py, not security validation.

**Finding**: There IS a security gap. The scan endpoint accepts any directory path. There is no restriction to configured allowed directories. However:
1. This is a local-only video editor (not a public web service)
2. The API is pre-v1.0 and designed for local use
3. Path traversal via `../` is mitigated by `Path.absolute()` resolution
4. The real risk is scanning unintended directories, not reading arbitrary files

**Conclusion**: R4 remains Medium severity. The audit (BL-025) should document this gap and recommend a configurable allowed-roots list for the Python layer. This is not a critical vulnerability for a local application but should be addressed. Scope creep risk is LOW — the fix is a simple config + check, not a Rust rewrite.

## I5: R6 — Rust Test Coverage Assessment

**Query**: Counted `#[test]` and `#[cfg(test)]` annotations across Rust source files.

**Finding**: 213 test-related annotations across 12 source files. The test modules are embedded in each source file (inline tests). Files with significant test counts: `range.rs` (51), `sanitize/mod.rs` (41), `filter.rs` (31), `validation.rs` (21), `ffmpeg/mod.rs` (14), `duration.rs` (13), `position.rs` (11), `timeline/mod.rs` (10).

**Assessment**: ~160+ individual tests across the Rust crate. All major modules have substantial test coverage. The 90% target in AGENTS.md is likely achievable — the codebase has good test discipline. Modules like `lib.rs` (2 tests) and `stub_gen.rs` (0 tests) may be below threshold but are small.

**Conclusion**: R6 severity confirmed Low. Baseline is likely 75-90% range. BL-010 should measure first, then set progressive threshold if needed.

## I6: U3 — FakeFFmpegExecutor Args Verification

**Query**: Read `src/stoat_ferret/ffmpeg/executor.py`, specifically FakeFFmpegExecutor (lines 171-248).

**Finding**: FakeFFmpegExecutor replays by sequential index (`self._index`). The `run()` method accepts `args` but only uses them for command reconstruction in the result (`command=["ffmpeg", *args]`). It does NOT validate args against the recorded args. The `stdin` and `timeout` parameters are explicitly "ignored".

**Recording format** (from RecordingFFmpegExecutor): Each recording stores `{"args": [...], "stdin": "...", "result": {...}}`. The args ARE recorded — they're just not checked during replay.

**Design recommendation**: Add an optional `strict` mode to FakeFFmpegExecutor that validates `args` match the recording's `args`. Default to non-strict for backward compatibility. Contract tests use strict mode.

**Conclusion**: U3 is well-understood. Implementation approach is clear — add optional args verification to FakeFFmpegExecutor.

## I7: U5 — llvm-cov CI Integration

**Query**: Read `.github/workflows/ci.yml`.

**Finding**: Current CI runs `cargo test` without coverage. The Rust test step is: `cargo test --manifest-path rust/stoat_ferret_core/Cargo.toml`. Adding llvm-cov requires: (1) installing `cargo-llvm-cov`, (2) replacing `cargo test` with `cargo llvm-cov`, (3) optionally uploading coverage report.

**CI structure**: The test job runs on a matrix (3 OS x 3 Python versions = 9 combinations). Rust coverage should only run on one combination (e.g., ubuntu-latest + Python 3.12) to avoid redundant work.

**Integration approach**: Add a `rust-coverage` step after `Rust tests` on ubuntu-latest only. Use `cargo-llvm-cov` with `--lcov --output-path lcov.info`. Upload as artifact. Threshold enforcement via `--fail-under-lines`.

**Conclusion**: U5 is well-understood. Standard pattern, no surprises expected.

## I8: R5 — DI Migration Override Points Verification

**Query**: Read `tests/test_api/conftest.py` dependency_overrides usage.

**Finding**: Exactly 4 override points confirmed at lines 129-132:
1. `get_repository` → `lambda: video_repository`
2. `get_project_repository` → `lambda: project_repository`
3. `get_clip_repository` → `lambda: clip_repository`
4. `get_video_repository` → `lambda: video_repository`

Note: `get_repository` and `get_video_repository` both map to the same `video_repository`. This appears to be a naming inconsistency (possibly legacy vs current naming).

**Conclusion**: R5 confirmed Low risk. 4 override points, well-understood. Migration is straightforward — add optional parameters to `create_app()` for each dependency.
