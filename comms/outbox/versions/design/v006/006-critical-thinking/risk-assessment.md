# Risk Assessment — v006 Effects Engine Foundation

## Risk: Clip Effect Model Design

- **Original severity**: High
- **Category**: Investigate now
- **Investigation**: Queried clip model in Python (`db/models.py:59-109`), Rust (`clip/mod.rs:63-76`), API schemas (`api/schemas/clip.py`), database schema (`db/schema.py:96-115`), and repository layer (`db/clip_repository.py`). Searched for existing JSON column patterns and migration mechanisms.
- **Finding**: The clip model has 8 scalar fields across all layers (Python, Rust, API). No effects field exists anywhere. The database uses raw SQLite with manual schema in `schema.py` — no Alembic migrations. However, an existing JSON-in-TEXT pattern exists in `db/audit.py` (`changes_json` field serialized via `json.dumps()`). The Rust clip struct does NOT need effects — effects are Python-side business logic that generates filter strings via Rust builders.
- **Resolution**: Add `effects_json TEXT` column to clips table in `schema.py`. Add `effects: list[dict[str, Any]] | None = None` to Python Clip dataclass. Add effects to ClipCreate/ClipUpdate/ClipResponse API schemas. Repository layer handles JSON serialization following the audit.py pattern. Rust clip struct remains unchanged — effects storage is Python-only; Rust generates filter strings from effect parameters. No migration needed since the project uses `CREATE TABLE IF NOT EXISTS` (fresh DBs get the new column; existing dev DBs can be recreated).
- **Affected themes/features**: Theme 03, Feature `002-clip-effect-api` (BL-043)

## Risk: FilterGraph Backward Compatibility

- **Original severity**: Medium
- **Category**: Investigate now
- **Investigation**: Searched all FilterGraph and FilterChain consumers across Rust, Python, and tests. Read `filter.rs:502-614` (FilterGraph implementation), `command.rs:418-439` (FFmpegCommand integration), `lib.rs:74` (PyO3 export), `test_pyo3_bindings.py:268-287` (Python test). Analyzed all 7 Rust tests and 1 Python test for FilterGraph usage patterns.
- **Finding**: All current FilterGraph consumers construct well-formed graphs with proper label wiring. Zero instances of invalid graph construction found in the codebase. FilterGraph currently has no validation — invalid graphs render to strings that FFmpeg rejects at runtime. `FFmpegCommand.filter_complex()` accepts a string (not a FilterGraph), so validation is decoupled from command building. The 004-research impact analysis explicitly recommends a separate `validate()` method.
- **Resolution**: Use opt-in validation via a separate `validate()` method, preserving backward compatibility. Add `validated_to_string()` as a convenience method. Existing `to_string()` / `Display` remains unchanged — no breaking changes. New v006 code (composition system, builders) should use `validate()` internally. Both methods exposed via PyO3 bindings.
- **Affected themes/features**: Theme 01, Feature `002-graph-validation` (BL-038) and Feature `003-filter-composition` (BL-039)

## Risk: Rust Coverage Ratchet

- **Original severity**: Medium
- **Category**: Accept with mitigation
- **Investigation**: Read CI workflow (`.github/workflows/ci.yml`), checked current threshold (`--fail-under-lines 75`), reviewed `docs/design/01-roadmap.md` (M1.9 target: >90%), examined existing test files across all Rust modules.
- **Finding**: Current CI threshold is 75% (set during v004). Roadmap target is >90%. The gap is 15 percentage points. v006 adds significant new Rust code (expression engine, graph validation, composition, drawtext, speed builders — 5+ new modules). New code will be well-tested by design (proptest for expressions, validation tests for graph, builder pattern tests). However, achieving 90% across the entire crate depends on existing module coverage too.
- **Resolution**: Do NOT attempt to ratchet to 90% during v006. Instead: (1) write comprehensive tests for all new v006 Rust code targeting >90% coverage on new modules, (2) keep CI threshold at 75% to avoid blocking v006 on legacy coverage gaps, (3) add a backlog item for a dedicated coverage improvement effort post-v006. The ratchet should happen in a version focused on test infrastructure, not during a feature-heavy version.
- **Affected themes/features**: All themes (all new Rust code should have high coverage)

## Risk: Font File Platform Dependency

- **Original severity**: Low
- **Category**: Accept with mitigation
- **Investigation**: Searched codebase for `fontfile`, `fontconfig`, `font.*path`, `system.*font`. No font-related code exists in the codebase. Reviewed FFmpeg drawtext documentation — `fontfile` is required without fontconfig; `font` parameter works when fontconfig is available.
- **Finding**: This is a runtime concern, not a design concern. FFmpeg's `font` parameter (as opposed to `fontfile`) uses fontconfig if available and is cross-platform. The drawtext builder should support both parameters. The builder API is a string generator — it doesn't resolve font paths itself. Font resolution is the caller's responsibility.
- **Resolution**: The drawtext builder (BL-040) should accept both `font(name)` and `fontfile(path)` as builder methods. Use `font` as the primary API (fontconfig-based, cross-platform). Document that `fontfile` is a fallback for systems without fontconfig. Tests should use `font="monospace"` or similar generic name. No platform detection logic in the builder — delegate to FFmpeg's existing fontconfig integration.
- **Affected themes/features**: Theme 02, Feature `001-drawtext-builder` (BL-040)

## Risk: boxborderw FFmpeg Version Dependency

- **Original severity**: Low
- **Category**: Accept with mitigation
- **Investigation**: Searched for FFmpeg version references in CI and tests. Found: test mocks use FFmpeg 6.0 (`test_health.py:27`), prototype design references 5.1.2 (`03-prototype-design.md:831`), health endpoint parses `ffmpeg -version` output. No formal minimum version policy exists.
- **Finding**: The multi-value `boxborderw=10|20|30|40` syntax requires FFmpeg 5.0+. All test references in the project use FFmpeg 5.1+ or 6.0. CI uses `setup-ffmpeg` action which installs a recent version. In practice, the project already targets FFmpeg 5.0+ implicitly.
- **Resolution**: Use only the single-value `boxborderw=N` syntax in v006 (uniform border on all sides). This works on all FFmpeg versions. Defer per-side border support to a future version if needed. Document the FFmpeg 5.0+ requirement for multi-value syntax in the builder's API docs. The drawtext builder should accept `boxborderw(width: u32)` as a single value — no pipe-delimited syntax.
- **Affected themes/features**: Theme 02, Feature `001-drawtext-builder` (BL-040)
