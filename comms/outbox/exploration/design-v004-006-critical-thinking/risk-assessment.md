# Risk Assessment — v004 Task 006

## Risk Triage Summary

| ID | Category | Resolution |
|----|----------|-----------|
| R1 | Investigate now | **Resolved** — no external consumers |
| R2 | Investigate now | **Resolved** — flat dataclasses, trivial overhead |
| R3 | Investigate now | **Resolved** — gap characterized, mitigation clear |
| R4 | Investigate now | **Resolved** — gap confirmed, bounded scope |
| R5 | Accept with mitigation | 4 overrides verified, low risk |
| R6 | Investigate now | **Resolved** — ~160 tests, likely 75-90% baseline |
| R7 | Accept with mitigation | Multi-stage builds, acceptable tradeoff |
| R8 | Accept with mitigation | Trivial — dev dependency only |
| U1 | TBD — runtime testing | Default 5-min timeout, configurable |
| U2 | TBD — runtime testing | Candidates identified, profiling at impl time |
| U3 | Investigate now | **Resolved** — optional strict mode for FakeFFmpegExecutor |
| U4 | TBD — runtime testing | python:3.12-slim + rustup as starting point |
| U5 | Investigate now | **Resolved** — standard cargo-llvm-cov pattern |

## Detailed Assessments

### R1: Scan API Breaking Change (BL-027)

- **Original severity**: High
- **Revised severity**: Low
- **Investigation**: Searched all files referencing `videos/scan`. Only consumers are: test suite (`tests/test_api/test_videos.py`), the endpoint itself, and design docs.
- **Finding**: No external consumers. Pre-v1.0 API with no GUI, CLI, or integrations.
- **Resolution**: Breaking change is safe. Test migration is the only required work.
- **Affected themes**: Theme 03 (async-scan) — no design changes needed.

### R2: Deepcopy Isolation Performance (BL-020)

- **Original severity**: Medium
- **Revised severity**: Low
- **Investigation**: Examined `Project` (7 scalar fields) and `Video` (14 scalar fields) dataclasses.
- **Finding**: All domain objects are flat dataclasses with scalar fields only (str, int, datetime). No nested objects, lists, or references. Deepcopy overhead is microseconds per object.
- **Resolution**: Use `copy.deepcopy()` without concern. No need for `copy.copy()` optimization.
- **Affected themes**: Theme 01 — no design changes needed.

### R3: FTS5 Emulation Fidelity (BL-016)

- **Original severity**: Medium
- **Revised severity**: Low
- **Investigation**: Compared SQLite FTS5 MATCH (`'"{query}"*'`) vs InMemory substring (`in`).
- **Finding**: Primary gap is substring vs prefix matching. FTS5 does prefix match; InMemory does substring. Multi-word phrase handling differs. Case handling is equivalent.
- **Resolution**: Per-token `startswith` matching closes the primary gap. Known FTS5 features not used by the app (AND/OR/NOT/NEAR/ranking) do not need emulation. Document known limitations.
- **Affected themes**: Theme 02 feature 030 — no structural changes, but implementation should document 3 known behavioral differences.

### R4: Security Audit Scope Creep (BL-025)

- **Original severity**: Medium (scope creep risk)
- **Revised severity**: Low (scope creep risk)
- **Investigation**: Rust `validate_path` only checks empty/null. Python `scan_directory` has no directory allowlist or traversal protection.
- **Finding**: Gap is real but bounded. The fix is a simple config-based allowed-roots check in the Python layer. Rust `validate_path` doesn't need modification — the deferred-to-Python comment is architecturally correct. No critical vulnerability for a local application.
- **Resolution**: BL-025 audit will recommend adding `ALLOWED_SCAN_ROOTS` to settings + check in scan service. This is ~20 lines of Python, not scope creep. Critical remediation scope is predictable.
- **Affected themes**: Theme 04 — no structural changes. Scope remains bounded.

### R5: Existing Test Breakage from DI Migration (BL-021)

- **Original severity**: Medium
- **Revised severity**: Low
- **Investigation**: Confirmed 4 dependency override points in conftest.py (lines 129-132).
- **Finding**: `get_repository` and `get_video_repository` both map to same instance (possible legacy naming). `create_app()` currently takes no parameters.
- **Mitigation**: Add optional params to `create_app()` defaulting to `None`. When `None`, use production defaults. Test conftest migrates from overrides to `create_app(video_repository=inmem, ...)`. Existing tests pass unchanged.
- **Affected themes**: Theme 01 feature 020 — well-understood migration path.

### R6: Rust Coverage Baseline Unknown (BL-010)

- **Original severity**: Low
- **Revised severity**: Low
- **Investigation**: Counted ~160+ `#[test]` functions across 12 Rust source files. All major modules have embedded test suites.
- **Finding**: Substantial test coverage exists. `range.rs` (51), `sanitize/mod.rs` (41), `filter.rs` (31), `validation.rs` (21) are well-tested. Likely 75-90% baseline.
- **Resolution**: Measure with llvm-cov first. If below 90%, set progressive threshold (e.g., 80% initial, ratchet up).
- **Affected themes**: Theme 05 feature 020 — no design changes needed.

### R7: Docker Image Complexity (BL-014)

- **Severity**: Low (unchanged)
- **Mitigation**: Multi-stage Dockerfile: Stage 1 builds Rust toolchain + maturin, Stage 2 is Python runtime. Cache Rust compilation layer. Expected 2-5 min build, 2-4 GB image. Use `python:3.12-slim` base with `rustup` install.
- **Affected themes**: Theme 05 feature 040.

### R8: Hypothesis Dependency Addition (BL-009)

- **Severity**: Low (unchanged)
- **Mitigation**: Add `hypothesis` as dev dependency with version pin in `pyproject.toml`.
- **Affected themes**: Theme 05 feature 010.

### U1: Job Queue Timeout Values (BL-027)

- **Category**: TBD — requires runtime testing
- **Current plan**: Default 5-minute timeout, configurable per job. Test with real media directories at implementation time.
- **Affected themes**: Theme 03.

### U2: Benchmark Baseline Operations (BL-026)

- **Category**: TBD — requires runtime testing
- **Current plan**: Candidates: timeline position arithmetic, filter string escaping, path validation. Profile at implementation time.
- **Affected themes**: Theme 04.

### U3: FakeFFmpegExecutor Args Verification (BL-024)

- **Category**: Investigated — design determined
- **Investigation**: FakeFFmpegExecutor records args but ignores them during replay.
- **Resolution**: Add optional `strict=False` parameter. When `strict=True`, verify `args` match recording. Contract tests use strict mode. Default non-strict preserves backward compatibility.
- **Affected themes**: Theme 02 feature 020.

### U4: Docker Base Image Selection (BL-014)

- **Category**: TBD — requires trial builds
- **Current plan**: Start with `python:3.12-slim` + `rustup`. Evaluate if pre-built images exist.
- **Affected themes**: Theme 05 feature 040.

### U5: llvm-cov CI Integration (BL-010)

- **Category**: Investigated — approach determined
- **Investigation**: CI uses matrix (3 OS x 3 Python). Rust coverage should run on one combo only.
- **Resolution**: Add `cargo-llvm-cov` step on ubuntu-latest + Python 3.12. Upload lcov report as artifact. Enforce threshold with `--fail-under-lines`.
- **Affected themes**: Theme 05 feature 020.
