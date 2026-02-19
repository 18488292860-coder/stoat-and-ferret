# Draft Checklist - v007 Document Drafts

- [x] manifest.json is valid JSON with all required fields
- [x] Every theme in manifest has a corresponding folder under drafts/
- [x] Every feature in manifest has a corresponding folder with both requirements.md and implementation-plan.md
- [x] VERSION_DESIGN.md and THEME_INDEX.md exist in drafts/
- [x] THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+`
- [x] No placeholder text in any draft (`_Theme goal_`, `_Feature description_`, `[FILL IN]`, `TODO`)
- [x] All backlog IDs from manifest appear in at least one requirements.md
- [x] No theme or feature slug starts with a digit prefix (`^\d+-`)
- [x] Backlog IDs in each requirements.md cross-referenced against Task 002 backlog analysis (no mismatches)
- [x] All "Files to Modify" paths verified via Glob queries against actual codebase (no unverified paths)

## Verification Details

**manifest.json**: Valid JSON, 4 themes, 11 features, 9 backlog IDs (BL-044 through BL-052).

**Folder structure**: 4 theme folders with THEME_DESIGN.md each, 11 feature folders with requirements.md + implementation-plan.md each. VERSION_DESIGN.md and THEME_INDEX.md present.

**THEME_INDEX format**: All 11 feature lines match `- \d{3}-[\w-]+: .+` regex pattern.

**No placeholders**: Scanned all 28 .md files — no instances of `_Theme goal_`, `_Feature description_`, `[FILL IN]`, or `TODO`.

**Backlog coverage**: All 9 backlog IDs (BL-044 through BL-052) appear in their corresponding feature requirements.md files.

**Slug format**: No theme or feature slug starts with a digit prefix.

**Backlog ID cross-reference**: Each BL number verified against Task 002 backlog-details.md:
- BL-044 → audio-mixing-builders (amix/volume/fade): Correct
- BL-045 → transition-filter-builders (fade/xfade): Correct
- BL-046 → transition-api-endpoint (transition API): Correct
- BL-047 → effect-registry-refactor (registry/schema): Correct
- BL-048 → effect-catalog-ui (catalog component): Correct
- BL-049 → dynamic-parameter-forms (form generator): Correct
- BL-050 → live-filter-preview (preview panel): Correct
- BL-051 → effect-builder-workflow (builder workflow): Correct
- BL-052 → e2e-effect-workshop-tests (E2E tests): Correct

**File path verification**: All "Files to Modify" paths verified via Glob queries against the actual codebase. Verified directories:
- `rust/stoat_ferret_core/src/ffmpeg/` — exists (parent for new audio.rs, transitions.rs)
- `rust/stoat_ferret_core/src/ffmpeg/mod.rs` — exists
- `rust/stoat_ferret_core/src/lib.rs` — exists
- `rust/stoat_ferret_core/src/sanitize/mod.rs` — exists
- `src/stoat_ferret/effects/definitions.py` — exists
- `src/stoat_ferret/effects/registry.py` — exists
- `src/stoat_ferret/api/routers/effects.py` — exists
- `src/stoat_ferret/api/schemas/effect.py` — exists
- `src/stoat_ferret/db/models.py` — exists
- `src/stoat_ferret/db/clip_repository.py` — exists
- `src/stoat_ferret/ffmpeg/metrics.py` — exists
- `stubs/stoat_ferret_core/_core.pyi` — exists
- `gui/src/App.tsx` — exists
- `gui/src/components/Navigation.tsx` — exists
- `gui/src/pages/` — exists (parent for new EffectsPage.tsx)
- `gui/src/stores/` — exists (parent for new stores)
- `gui/src/hooks/` — exists (parent for new hooks)
- `gui/src/components/` — exists (parent for new components)
- `gui/e2e/accessibility.spec.ts` — exists
- `docs/design/02-architecture.md` — exists
- `docs/design/05-api-specification.md` — exists
- `docs/design/01-roadmap.md` — exists
- `docs/design/08-gui-architecture.md` — exists
- `tests/test_api/test_effects.py` — exists
- `tests/test_pyo3_bindings.py` — exists
