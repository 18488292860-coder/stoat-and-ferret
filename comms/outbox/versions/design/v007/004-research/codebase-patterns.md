# Codebase Patterns - v007 Research

## Rust Builder Pattern (Template for BL-044, BL-045)

**Files**: `rust/stoat_ferret_core/src/ffmpeg/drawtext.rs`, `speed.rs`

The proven builder template from v006 (LRN-028):
1. `#[pyclass]` struct with typed fields and sensible defaults
2. `#[new]` constructor with validation (returns `PyResult`)
3. Fluent methods returning `PyRefMut<'_, Self>` for Python chaining (LRN-001)
4. `.build()` method returning `Filter` struct
5. PyO3 `#[pymethods]` block with `#[pyo3(name = "...")]` for clean Python API
6. `#[gen_stub_pyclass]` for stub generation

**DrawtextBuilder** (drawtext.rs:144-645): 10 builder methods, Position enum with 7 variants, text escaping. Single-input filter.

**SpeedControl** (speed.rs:98-166): Simple struct, `.setpts_filter()` + `.atempo_filters()` returning separate Filter objects. Audio chaining splits factors outside [0.5, 2.0] range.

**Key pattern for new builders**: All validation happens in Rust (`validate_volume` in sanitize/mod.rs:319-356, range 0.0-10.0). Python layer does business logic only (LRN-011).

## Two-Input Filter Pattern (New for v007)

**FilterChain** (filter.rs:433-531) already supports multiple inputs:
```
FilterChain::new().input("0:v").input("1:v").filter(xfade_filter).output("out")
```
The `.input()` method (line 475) adds input pad labels. xfade and sidechaincompress will use this existing mechanism. No FilterChain changes needed.

## Effect Registry (Refactoring Target for BL-047)

**EffectRegistry** (`src/stoat_ferret/effects/registry.py:12-50`): Plain dict wrapper with `register()`, `get()`, `list_all()`. Stores `EffectDefinition` instances.

**EffectDefinition** (`src/stoat_ferret/effects/definitions.py:15-32`): Frozen dataclass with `name`, `description`, `parameter_schema`, `ai_hints`, `preview_fn`. Currently missing a `build_fn` for filter string generation.

**_build_filter_string()** (`src/stoat_ferret/api/routers/effects.py:98-142`): if/elif dispatch over effect_type string. This is the documented refactoring trigger from v006 — "refactor to registry dispatch when 3rd effect type added" (LRN-029). BL-047 adds 3+ types.

**Refactoring approach**: Add `build_fn: Callable[[dict[str, Any]], str]` to `EffectDefinition`. Each effect registers its own builder. Replace `_build_filter_string()` with `definition.build_fn(parameters)`. This is backward-compatible — existing definitions just gain a new field.

## DI Pattern (Extension Point for BL-047)

**create_app()** (`src/stoat_ferret/api/app.py:91-169`): Already accepts `effect_registry` as optional kwarg (line 98). Stored on `app.state.effect_registry` (line 142). Retrieved via `get_effect_registry()` dependency (effects.py:32-51) with fallback to `create_default_registry()`.

No changes to the DI pattern needed. New effect types register in `create_default_registry()`.

## Effects Storage (Extension Point for BL-051)

**Clip.effects** (`src/stoat_ferret/db/models.py:75`): `list[dict[str, Any]] | None`. Stored as `effects_json TEXT` column in SQLite.

**Effect entry** (effects.py:218-222): `{"effect_type": str, "parameters": dict, "filter_string": str}`. Append-only — no update/delete operations exist yet.

**BL-051 needs**: Edit = replace entry at index. Remove = pop entry at index. Reorder = swap indices. All operations on the Python list, then persist via `clip_repo.update(clip)`.

## Prometheus Metrics (Extension Point for BL-047)

**Existing**: `http_requests_total` Counter, `http_request_duration_seconds` Histogram (middleware/metrics.py:15-25). FFmpeg metrics in ffmpeg/metrics.py:11-26.

**BL-047 needs**: `effect_applications_total` Counter with `effect_type` label. Add in effects router, increment on successful apply. Same pattern as existing counters.

## GUI Component Patterns (Template for BL-048-051)

**Zustand stores** (gui/src/stores/): 3 focused stores (LRN-024 verified). Pattern: `create<State>((set) => ({...}))` with setter functions. Effect workshop needs 3-4 new stores: catalog, form params, preview, effect stack.

**Custom hooks** (gui/src/hooks/): Data fetching hooks with `loading`/`error`/`refetch` pattern. Effect catalog hook will follow `useProjects` pattern.

**Routing** (gui/src/App.tsx): React Router with `/gui` basename. Effect workshop needs new route(s). SPA fallback still missing (LRN-023 verified).

**E2E tests** (gui/e2e/): Playwright with axe-core accessibility. Navigate via client-side clicks, not direct URLs (LRN-023). BL-052 follows this pattern.

## Learning Verification Table

| Learning | Status | Evidence |
|----------|--------|----------|
| LRN-001 | VERIFIED | PyRefMut chaining pattern present in drawtext.rs:436-645, speed.rs |
| LRN-005 | VERIFIED | create_app() accepts effect_registry kwarg (app.py:98), stored on app.state (line 142) |
| LRN-011 | VERIFIED | validate_volume in Rust (sanitize/mod.rs:319-356), business logic in Python effects.py |
| LRN-019 | VERIFIED | Meta-pattern; v006 validated infrastructure-first sequencing |
| LRN-024 | VERIFIED | 3 focused Zustand stores: projectStore, libraryStore, activityStore |
| LRN-026 | VERIFIED | FilterGraph.validate() and .validated_to_string() exist alongside .to_string() (filter.rs:686-799) |
| LRN-028 | VERIFIED | Template proven across 5 v006 features; DrawtextBuilder and SpeedControl follow identical pattern |
| LRN-029 | VERIFIED | _build_filter_string() still uses if/elif dispatch (effects.py:111-142), effects_json is TEXT column |
| LRN-030 | VERIFIED | Meta-pattern; v006 T03 F003 architecture docs feature validated approach |
| LRN-031 | VERIFIED | Meta-pattern; v006 achieved 100% first-iteration success with detailed AC |
| LRN-022 | VERIFIED | axe-core accessibility tests in gui/e2e/accessibility.spec.ts |
| LRN-023 | VERIFIED | E2E tests navigate via clicks from /gui/; SPA fallback routing still missing |
