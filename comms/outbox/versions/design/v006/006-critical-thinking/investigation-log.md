# Investigation Log — v006 Critical Thinking

## Investigation 1: Clip Effect Model Design

**Hypothesis:** The clip model needs an effects field added across Python model, API schemas, database schema, and repository — and the Rust clip struct may also need changes.

**Queries performed:**
1. Read `src/stoat_ferret/db/models.py:59-109` — Clip dataclass with 8 scalar fields, no effects
2. Read `rust/stoat_ferret_core/src/clip/mod.rs:63-76` — Rust Clip struct with source_path, in/out points, no effects
3. Read `src/stoat_ferret/api/schemas/clip.py` — Pydantic schemas for ClipCreate/Update/Response, no effects
4. Read `src/stoat_ferret/db/schema.py:96-115` — Raw SQL CREATE TABLE for clips, no JSON columns
5. Read `src/stoat_ferret/db/clip_repository.py:96-174` — Repository layer, scalar field handling only
6. Read `src/stoat_ferret/db/audit.py:5,53` — Found existing JSON-in-TEXT pattern with `changes_json`
7. Read `src/stoat_ferret/api/app.py:90-162` — DI pattern via `create_app()` kwargs
8. Searched for migration tooling (Alembic) — none found; schema is static SQL in `schema.py`

**Key evidence:**
- `schema.py` uses `CREATE TABLE IF NOT EXISTS` — fresh databases get schema from code
- No Alembic or migration framework — schema changes are manual
- `audit.py:53` demonstrates JSON serialization: `json.dumps(changes) if changes else None`
- Rust clip struct handles validation (in_point < out_point, within source_duration) — not business logic
- Effects are business-level data (which effects are applied) → Python-side concern

**Conclusion:** Rust clip needs NO changes. Python model/schema/repository/API all need effects field. Follow audit.py JSON pattern. No migration framework needed — dev databases are ephemeral.

---

## Investigation 2: FilterGraph Backward Compatibility

**Hypothesis:** Adding validation to FilterGraph might break existing consumers that construct incomplete or unusual graphs.

**Queries performed:**
1. Read `rust/stoat_ferret_core/src/ffmpeg/filter.rs:502-614` — Full FilterGraph implementation
2. Grep for `FilterGraph` across entire codebase — 34 file matches
3. Analyzed all 7 Rust tests in `filter.rs:807-916` — all construct well-formed graphs
4. Read `tests/test_pyo3_bindings.py:268-287` — Python test constructs valid 3-chain graph
5. Read `rust/stoat_ferret_core/src/ffmpeg/command.rs:418-439` — `filter_complex()` accepts String, not FilterGraph
6. Read `rust/stoat_ferret_core/src/lib.rs:74` — PyO3 export of FilterGraph class
7. Read `comms/outbox/versions/design/v006/004-research/impact-analysis.md` — recommends separate `validate()` method

**Key evidence:**
- All existing tests and production code construct correctly-wired graphs
- Zero instances of intentionally invalid graph construction
- `FFmpegCommand.filter_complex()` accepts a string → decoupled from FilterGraph
- Task 004 impact analysis explicitly recommends opt-in validation
- Labels stored as `Vec<String>` — no type distinction between stream refs (0:v) and inter-chain labels

**Conclusion:** Opt-in `validate()` method is safe — zero breaking changes. All existing code produces valid graphs. Validation catches future mistakes without disrupting current code.

---

## Investigation 3: Rust Coverage Threshold

**Hypothesis:** The 75% → 90% coverage gap may be closeable during v006 since we're adding extensively-tested new code.

**Queries performed:**
1. Read `.github/workflows/ci.yml` — `--fail-under-lines 75` for Rust
2. Read `docs/design/01-roadmap.md:118` — M1.9 target: >90% Rust coverage
3. Searched for test files in `rust/stoat_ferret_core/` — found tests in timeline, clip, ffmpeg, sanitize modules
4. Analyzed existing proptest usage in `timeline/mod.rs` and `timeline/range.rs`
5. Reviewed PLAN.md v004 entry — notes "Rust coverage threshold at 75% (target 90%)" as deferred

**Key evidence:**
- v004 explicitly deferred the 90% ratchet (PLAN.md line 89)
- Current 75% threshold set during v004 testing infrastructure version
- Existing pure function architecture supports high coverage
- v006 adds 5+ new Rust modules — all will have tests by design
- The 15% gap requires both new code tests AND legacy code improvement

**Conclusion:** Don't ratchet to 90% during v006. New code should target >90% individually. The ratchet is a separate effort, not a blocker for this version.

---

## Investigation 4: Font File Platform Dependency

**Hypothesis:** The drawtext `fontfile` parameter creates a platform dependency that the builder must handle.

**Queries performed:**
1. Grep for `fontfile`, `fontconfig`, `font.*path`, `system.*font` — no matches in production code
2. Reviewed FFmpeg drawtext documentation (004-research/external-research.md Section 2)
3. Reviewed CI configuration for OS matrix — tests run on ubuntu/macos/windows

**Key evidence:**
- Zero font-related code in codebase — drawtext is greenfield
- FFmpeg `font` parameter uses fontconfig (cross-platform) vs `fontfile` (absolute path)
- The builder generates filter strings — it doesn't resolve font paths
- Cross-platform CI means tests must use portable font references

**Conclusion:** Builder accepts both `font()` and `fontfile()`. Tests use `font="monospace"` (fontconfig). No platform detection needed in the builder.

---

## Investigation 5: boxborderw FFmpeg Version

**Hypothesis:** The multi-value `boxborderw` syntax might be needed for v006, creating an FFmpeg version dependency.

**Queries performed:**
1. Grep for `ffmpeg.*version` across codebase — found version references in tests and docs
2. Read `tests/test_api/test_health.py:27` — mocks FFmpeg 6.0
3. Read `docs/design/03-prototype-design.md:831` — references FFmpeg 5.1.2
4. Checked CI for `setup-ffmpeg` action

**Key evidence:**
- All version references are 5.0+ (test mocks: 6.0, prototype: 5.1.2)
- CI uses `setup-ffmpeg` action (installs recent FFmpeg)
- Multi-value `boxborderw` is a convenience feature, not required for v006 functionality
- Single-value `boxborderw=N` works on all FFmpeg versions

**Conclusion:** Use single-value `boxborderw` only in v006. Defer per-side borders. No version policy change needed.
