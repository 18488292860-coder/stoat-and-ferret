# Wiring Audit: v001 & v002

**7 wiring gaps found** across v001 and v002. All gaps are in v002 themes 03 (database-foundation) and 04 (ffmpeg-integration). v001 and v002 themes 01-02 are clean.

The pattern identified in the known issue (configure_logging created but never called, settings.log_level defined but never consumed) is replicated across multiple v002 features. Quality gates passed because unit tests validated individual pieces in isolation — no integration test checks whether startup actually wires everything together.

## Per-Theme Findings

### v001/01-project-foundation — Clean
All 3 features (python-tooling, rust-workspace, ci-pipeline) are foundational infrastructure and fully wired.

### v001/02-timeline-math — Cosmetic
Position, Duration, FrameRate are used in production (clip validation). TimeRange and its list operations (find_gaps, merge_ranges, total_coverage) are fully implemented and exposed but have zero production callers — design-ahead code for future timeline editing features.

### v001/03-command-builder — Cosmetic
FFmpegCommand, Filter, FilterChain, FilterGraph are all wired and used. Input sanitization functions (validate_crf, validate_speed, etc.) are exposed via PyO3 but never called from Python production code. Internal Rust validation in FFmpegCommand.build() covers overlapping checks.

### v002/01-rust-python-bindings — Clean
All 4 features (stub-regeneration, clip-bindings, range-list-ops, api-naming-cleanup) are properly wired with correct PyO3 naming, stubs, and CI verification.

### v002/02-tooling-process — Clean
Documentation-only feature (agents-pyo3-guidance). AGENTS.md properly updated.

### v002/03-database-foundation — Critical gaps
- **create_tables() never called at startup** (critical) — fresh database gets no schema
- **Alembic migrations never run at startup** (critical) — manual CLI step required
- **AuditLogger never instantiated** (minor) — parameter exists on repository but always None

### v002/04-ffmpeg-integration — Critical gaps
- **configure_logging() never called** (critical) — structured logging config is dead code
- **ObservableFFmpegExecutor never instantiated** (critical) — metrics/observability layer unused
- **settings.log_level never consumed** (minor) — uvicorn uses hardcoded "info"
- **execute_command() never used** (minor) — integration bridge between Rust builder and Python executor has no callers

### Orphaned Settings
3 settings in settings.py are defined but never consumed by application code:
- `debug` — never referenced
- `log_level` — never passed to configure_logging (which itself is never called)
- `ws_heartbeat_interval` — hardcoded to 30 in ws.py instead of reading from settings
