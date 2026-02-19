# Impact Analysis - v007

## Dependencies: Existing Code That Will Change

### Rust Crate (`rust/stoat_ferret_core/`)

| File | Change | Caused By |
|------|--------|-----------|
| `src/ffmpeg/mod.rs` | Add `audio`, `transitions` submodules and re-exports | BL-044, BL-045 |
| `src/lib.rs` | Register new PyO3 classes (audio builders, transition builders, enums) | BL-044, BL-045 |
| `src/sanitize/mod.rs` | Potentially add `validate_fade_duration()`, `validate_transition_type()` | BL-044, BL-045 |
| NEW `src/ffmpeg/audio.rs` | AmixBuilder, VolumeBuilder, AfadeBuilder, DuckingPattern | BL-044 |
| NEW `src/ffmpeg/transitions.rs` | FadeBuilder, XfadeBuilder, TransitionType enum | BL-045 |

### Python Source (`src/stoat_ferret/`)

| File | Change | Caused By |
|------|--------|-----------|
| `effects/definitions.py` | Add EffectDefinition instances for audio/transition effects; add `build_fn` field | BL-044, BL-045, BL-047 |
| `effects/registry.py` | May gain JSON schema validation method; builder protocol | BL-047 |
| `api/routers/effects.py` | Replace `_build_filter_string()` with registry dispatch; add CRUD endpoints; add transition endpoint | BL-046, BL-047, BL-051 |
| `api/schemas/effect.py` | Add transition request/response schemas; effect CRUD schemas | BL-046, BL-051 |
| `api/app.py` | No changes expected (existing DI pattern sufficient) | — |
| `api/middleware/metrics.py` or new metrics file | Add `effect_applications_total` Counter | BL-047 |

### Frontend (`gui/src/`)

| Location | Change | Caused By |
|----------|--------|-----------|
| `App.tsx` | Add route for effect workshop | BL-048-051 |
| `components/Navigation.tsx` | Add "Effects" tab | BL-048-051 |
| NEW `stores/effectCatalogStore.ts` | Catalog state (search, filter, selected) | BL-048 |
| NEW `stores/effectFormStore.ts` | Form parameter state, validation errors | BL-049 |
| NEW `stores/effectPreviewStore.ts` | Filter string preview state | BL-050 |
| NEW `stores/effectStackStore.ts` | Per-clip effect stack state | BL-051 |
| NEW `hooks/useEffects.ts` | Fetch effects from API | BL-048 |
| NEW `hooks/useEffectPreview.ts` | Debounced preview API calls | BL-050 |
| NEW `components/EffectCatalog.tsx` | Grid/list view, search, filter | BL-048 |
| NEW `components/EffectParameterForm.tsx` | Schema-driven form generator | BL-049 |
| NEW `components/FilterPreview.tsx` | Filter string display with highlighting | BL-050 |
| NEW `components/EffectStack.tsx` | Effect list per clip with edit/remove | BL-051 |
| NEW `components/ClipSelector.tsx` | Clip selection for effect application | BL-051 |
| NEW `pages/EffectsPage.tsx` | Workshop page composing all components | BL-051 |

### Stubs (`stubs/stoat_ferret_core/`)

| File | Change | Caused By |
|------|--------|-----------|
| `_core.pyi` | Add audio builder classes, transition builder classes, TransitionType enum | BL-044, BL-045 |

## Breaking Changes

### API Breaking Changes: None

All changes are additive:
- New endpoints (POST /effects/transition, PATCH/DELETE effects) don't affect existing endpoints
- Existing `GET /effects` response gains additional effects but existing effects remain unchanged
- EffectDefinition gains `build_fn` field but existing instances work with a default

### Internal Breaking Change: `_build_filter_string()` Removal

**What**: The if/elif dispatch function in `effects.py:98-142` will be replaced by registry-based dispatch.
**Impact**: All existing tests calling `_build_filter_string()` directly (if any) will need updating. The function is private (`_` prefix) so external consumers shouldn't exist.
**Mitigation**: Replace in BL-047 feature. All effect application tests go through the API endpoint, not the private function directly.

## Test Infrastructure Needs

### New Python Tests

| Test Area | Type | Estimated Count | For |
|-----------|------|-----------------|-----|
| Audio builder unit tests | Unit | ~15 | BL-044 |
| Transition builder unit tests | Unit | ~12 | BL-045 |
| Transition API endpoint tests | Integration | ~8 | BL-046 |
| Registry refactoring tests | Unit + Integration | ~10 | BL-047 |
| Effect CRUD endpoint tests | Integration | ~8 | BL-051 |

### New Rust Tests

| Test Area | Estimated Count | For |
|-----------|-----------------|-----|
| AmixBuilder tests | ~8 | BL-044 |
| VolumeBuilder tests | ~6 | BL-044 |
| AfadeBuilder tests | ~8 | BL-044 |
| DuckingPattern tests | ~5 | BL-044 |
| FadeBuilder tests | ~6 | BL-045 |
| XfadeBuilder tests | ~8 | BL-045 |
| TransitionType enum tests | ~3 | BL-045 |

### New Frontend Tests

| Test Area | Type | For |
|-----------|------|-----|
| EffectCatalog component tests | Vitest | BL-048 |
| EffectParameterForm component tests | Vitest | BL-049 |
| FilterPreview component tests | Vitest | BL-050 |
| EffectStack component tests | Vitest | BL-051 |
| ClipSelector component tests | Vitest | BL-051 |
| Effect workshop E2E tests | Playwright | BL-052 |
| Accessibility tests for forms | Playwright + axe-core | BL-052 |

### Existing Tests: No Breakage Expected

All existing tests (733+) should remain passing:
- New Rust modules don't modify existing modules
- Registry refactoring replaces a private function, API behavior unchanged
- New endpoints are additive
- Frontend routes are additive

## Documentation Updates Required

Per 003-impact-assessment, 13 documentation impacts consolidated into 2 features:

1. **Architecture Documentation Feature**: Update `docs/design/02-architecture.md` (7 impacts — Rust modules, Effects Service rewrite, PyO3 bindings, registered effects, metrics)
2. **API Specification Documentation Feature**: Update `docs/design/05-api-specification.md` (4 impacts — transition endpoint, preview endpoint, CRUD endpoints, discovery response)
3. **Milestone checkboxes**: `docs/design/01-roadmap.md` and `docs/design/08-gui-architecture.md` (2 trivial impacts)
