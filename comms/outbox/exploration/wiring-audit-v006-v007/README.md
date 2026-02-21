# Wiring Audit: v006 & v007

Four wiring gaps found across v006 and v007. Three are cosmetic (dead Python bindings for Rust code that works fine internally), one is minor (transition API has no GUI). No critical gaps — all registered effects resolve, all API endpoints are mounted, and the Effect Workshop GUI connects to real backend endpoints.

The known logging gap is excluded per the prompt.

## Per-Theme Findings

### v006 Theme 01: Filter Engine — 3 cosmetic gaps

All three features (Expression Engine, Graph Validation, Filter Composition) were implemented correctly in Rust with working PyO3 bindings. However, none of the Python bindings are used in production code — only in parity tests.

| Feature | Gap | Severity |
|---------|-----|----------|
| 001 Expression Engine | `Expr` class exposed to Python but never imported in production code. Used internally in Rust by `DrawtextBuilder.alpha_fade()`. | cosmetic |
| 002 Graph Validation | `validate()` and `validated_to_string()` exposed to Python but never called. Effects pipeline sends filter strings to FFmpeg without validation. | cosmetic |
| 003 Filter Composition | `compose_chain()`, `compose_branch()`, `compose_merge()` exposed to Python but never called. `DuckingPattern` uses composition internally in Rust. | cosmetic |

### v006 Theme 02: Filter Builders — clean

Both builders (`DrawtextBuilder`, `SpeedControl`) are imported and used in `effects/definitions.py`. No gap.

### v006 Theme 03: Effects API — clean

`EffectRegistry`, `GET /effects`, and `POST` clip effects endpoint all wired correctly. The handoff note about refactoring `_build_filter_string()` dispatch was actioned in v007 T02-F001.

### v007 Theme 01: Rust Filter Builders — clean

All 4 audio builders and 4 transition builders are imported in `effects/definitions.py` and registered with the effect registry. No gap.

### v007 Theme 02: Effect Registry API — 1 minor gap

| Feature | Gap | Severity |
|---------|-----|----------|
| 002 Transition API | `POST /projects/{id}/effects/transition` endpoint is implemented but no GUI calls it. The Effect Workshop handles per-clip effects only. | minor |

The registry refactor (F001) and architecture docs (F003) are clean.

### v007 Theme 03: Effect Workshop GUI — clean

All GUI API calls resolve to real backend endpoints. Effect catalog, parameter forms, live preview, and CRUD workflow are all wired. One handoff note about surfacing backend validation errors into form fields was not actioned (client-side HTML validation only).

### v007 Theme 04: Quality Validation — clean

E2E tests and docs updates are wired. Pre-existing flaky E2E test (BL-055) is a test infrastructure issue, not a wiring gap.
