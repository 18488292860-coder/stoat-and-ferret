# Retrospective Insights - v006 -> v007

## Source

Previous version: **v006** (Effects Engine Foundation)
Retrospective: `comms/outbox/versions/execution/v006/retrospective.md`
Theme retrospectives: `docs/versions/v006/0X-*_retrospective.md`

## What Worked Well (Continue)

### Infrastructure-First Sequencing Validated
The three-theme dependency chain (expression engine -> builders -> API) executed cleanly with zero rework. Each theme built on the previous one. **v007 implication:** Sequence Rust builders (audio, transitions) before registry, registry before GUI components, GUI components before workflow integration.

### Builder Pattern Proven Across 5 Features
The Rust builder -> PyO3 bindings -> type stubs -> Python tests pattern was validated across all v006 features. Each successive feature was faster. **v007 implication:** Audio mixing (BL-044) and transition (BL-045) builders should follow the identical template. No need to invent a new pattern.

### Opt-In Validation Preserves Compatibility
Making graph validation opt-in (`validate()` / `validated_to_string()`) meant all existing tests passed unchanged. **v007 implication:** New audio/transition validation should follow this pattern — add new validation methods alongside existing behavior.

### Single-Iteration Success (100%)
All 8 features across 3 themes passed on first iteration with zero quality gate failures, confirming that detailed design specifications lead to clean execution. **v007 implication:** Invest heavily in design quality. Detailed acceptance criteria with 4-6 testable items per feature.

### DI Pattern Scales Cleanly
Constructor DI via `create_app()` kwargs extended to `effect_registry` with zero friction. **v007 implication:** Effect registry enhancements and any new services should continue this pattern.

### Documentation as First-Class Feature
Architecture docs in theme 03 feature 003 caught discrepancies between design and implementation. **v007 implication:** Include architecture documentation feature in themes with API or structural changes.

## What Didn't Work (Avoid)

### Formulaic Use Cases on All Backlog Items
All 9 v007 backlog items have the identical formulaic use case pattern ("This feature addresses: {title}. It improves the system by resolving the described requirement."). This provides zero information beyond the title and risks scope invention during design. Quality refinement was not possible due to tool authorization constraints.

### Pre-Existing mypy Errors Accumulating
11 pre-existing mypy errors carried through all v006 features without resolution. **v007 implication:** Track and avoid increasing this count. Consider a maintenance pass if adding new typed code increases friction.

## Tech Debt Addressed in v006

- Expression engine, graph validation, composition API — all **addressed** (v006 T01)
- Text overlay and speed control builders — all **addressed** (v006 T02)
- Effect discovery and clip effect application — all **addressed** (v006 T03)
- C4 architecture documentation — **addressed** (v006 T03 F003 + post-version regeneration)

## Tech Debt Deferred (Relevant to v007)

| Item | Source | v007 Impact |
|------|--------|-------------|
| `_build_filter_string()` if/elif dispatch | v006 T03 | **Must address** — BL-047 registry adds 3rd+ effect type, triggering the documented refactor point |
| No effect CRUD (update/delete) endpoints | v006 T03 | **Must address** — BL-051 requires edit/remove of applied effects |
| No effect ordering/priority system | v006 T03 | **Should address** — BL-051 effect stack implies ordering |
| No effect preview endpoint (dry-run) | v006 T03 | **Should address** — BL-050 live preview may need this |
| SPA fallback routing | v005 | **Should address** — Effect workshop deep-links need SPA fallback |
| Rust coverage at 75% vs 90% target | v004 | **Track** — New Rust code should increase coverage |

## Architectural Decisions Informing v007

1. **Registry replaces if/elif dispatch:** v006 T03 explicitly noted "refactor to registry dispatch when 3rd added." BL-047 is that moment.
2. **EffectRegistry is extensible:** v006 T03 retrospective notes the existing `register()` API can support dynamic effect loading for the Effect Workshop.
3. **Rust builders as single source of truth:** Both discovery (preview_fn) and application (`_build_filter_string`) call the same builders. BL-050 live preview should follow this pattern.
4. **JSON column for effect storage:** Effects stored as `effects_json TEXT` works for append-only. BL-051's edit/remove may need schema evolution.
5. **Simple data structures are sufficient:** Plain dict wrappers and JSON columns are fine until complexity demands more. Apply this to BL-047 registry design.
