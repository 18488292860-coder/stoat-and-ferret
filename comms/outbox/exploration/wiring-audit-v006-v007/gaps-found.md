# Wiring Gaps: v006 & v007

## Gap 1: Expr PyO3 bindings unused from Python

- **Version/Theme/Feature**: v006 / T01 Filter Engine / F001 Expression Engine
- **Designed**: `Expr` class with 17 static methods, operator overloading, and PyO3 bindings for building FFmpeg filter expressions from Python
- **Wired up**: Rust implementation works. PyO3 bindings exist. 17 parity tests pass. But zero imports of `Expr` in `src/stoat_ferret/`. The class is only used internally in Rust (e.g., `DrawtextBuilder.alpha_fade()` generates `if(lt(t,...))` expressions).
- **Completion report caught it**: No. Reported all 5 acceptance criteria passed.
- **Severity**: cosmetic (dead code). The Rust-side expression engine works fine through the builders. The Python bindings are unnecessary overhead but cause no harm.

## Gap 2: FilterGraph validation not wired into effects pipeline

- **Version/Theme/Feature**: v006 / T01 Filter Engine / F002 Graph Validation
- **Designed**: `FilterGraph.validate()` and `FilterGraph.validated_to_string()` for detecting unconnected pads, cycles, and duplicate labels before sending to FFmpeg
- **Wired up**: Rust implementation works. PyO3 bindings exist. 6 parity tests pass. But no Python production code calls validation. The effects pipeline in `definitions.py` builds filter strings via builders and returns `str(builder.build())` without any validation step.
- **Completion report caught it**: No. Reported as complete with all criteria passed.
- **Severity**: cosmetic. Current builders produce valid graphs by construction. Validation would add defense-in-depth but isn't required for correctness with the current builder-only approach.

## Gap 3: FilterGraph composition API unused from Python

- **Version/Theme/Feature**: v006 / T01 Filter Engine / F003 Filter Composition
- **Designed**: `compose_chain()`, `compose_branch()`, `compose_merge()` on `FilterGraph` for programmatic multi-filter pipeline construction from Python
- **Wired up**: Rust implementation works. PyO3 bindings exist. 14 parity tests pass. But `FilterGraph` is never imported in `src/stoat_ferret/`. The `DuckingPattern` builder uses composition internally in Rust, and all other effects use single-filter builders.
- **Completion report caught it**: No. Reported as complete.
- **Severity**: cosmetic (dead code). The composition API is a building block for future multi-filter effects. Current effects don't need it because each maps to 1-2 FFmpeg filters.

## Gap 4: Transition API has no GUI

- **Version/Theme/Feature**: v007 / T02 Effect Registry API / F002 Transition API Endpoint
- **Designed**: `POST /projects/{id}/effects/transition` endpoint for applying crossfade/xfade transitions between adjacent clips, with GUI integration expected via the Effect Workshop
- **Wired up**: Backend endpoint is fully implemented — validates clip adjacency, generates filter string via registry dispatch, stores transition persistently. But the Effect Workshop GUI has no transition UI. All 7 GUI API calls target per-clip effect endpoints; none call the transition endpoint.
- **Completion report caught it**: No. The transition API completion report noted it was complete. The Effect Workshop completion reports focused on per-clip effects.
- **Severity**: minor (degraded). The transition feature works via API (curl/tests) but is not user-accessible through the GUI. Users cannot apply transitions without direct API calls.

## Unactioned Handoff Notes (not wiring gaps, but worth tracking)

1. **Backend validation → form fields** (v007 T03-F002 handoff): The dynamic parameter forms use HTML `min`/`max` attributes for client-side validation but don't surface backend validation errors into per-field error messages. Backend errors appear as generic toast/error messages.

2. **Alembic migration** (v007 T02-F002 handoff): The `transitions_json` column was added to the database schema but no Alembic migration was created for existing databases.
