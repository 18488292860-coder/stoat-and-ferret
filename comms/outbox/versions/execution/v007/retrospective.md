# v007 Version Retrospective

## Version Summary

**Version:** v007 — Effect Workshop GUI
**Started:** 2026-02-19
**Completed:** 2026-02-19
**Themes:** 4 (all completed)
**Features:** 11 (all completed)

v007 delivered the complete Effect Workshop: Rust filter builders for audio mixing and video transitions, a refactored effect registry with builder-protocol dispatch and JSON schema validation, the full GUI workshop (catalog, parameter forms, live preview, builder workflow), and end-to-end quality validation with accessibility compliance. This version covers roadmap milestones M2.4–M2.6, M2.8, and M2.9 (4/5 sub-items).

## Theme Results

| # | Theme | Features | Status | Acceptance Criteria | Key Outcome |
|---|-------|----------|--------|---------------------|-------------|
| 01 | rust-filter-builders | 2/2 | Complete | 42/42 | AmixBuilder, VolumeBuilder, AfadeBuilder, DuckingPattern, FadeBuilder, XfadeBuilder, AcrossfadeBuilder, TransitionType enum |
| 02 | effect-registry-api | 3/3 | Complete | 26/26 | Builder-protocol dispatch replacing if/elif, JSON schema validation, POST /effects/transition endpoint, architecture docs |
| 03 | effect-workshop-gui | 4/4 | Complete | 49/49 | EffectCatalog, EffectParameterForm, FilterPreview, ClipSelector, EffectStack components; 4 Zustand stores |
| 04 | quality-validation | 2/2 | Complete | 14/14 | 8 E2E Playwright tests, axe-core WCAG AA compliance, API spec/roadmap/GUI architecture docs updated |

**Total acceptance criteria:** 131/131 passed

## C4 Documentation

**Status:** Successfully regenerated

C4 architecture documentation was regenerated at all levels (code, component, container, context) following v007 completion. No issues encountered.

## Cross-Theme Learnings

### Builder pattern scales well (Themes 01–02)
The v006 builder template pattern (LRN-028) transferred cleanly to audio mixing, transitions, and then into the registry's builder-protocol dispatch. Seven new Rust builder classes were added with consistent fluent API ergonomics. The pattern is now proven across 9 registered effects.

### Schema-driven architecture from backend to frontend (Themes 02–03)
JSON schemas defined in the effect registry (Theme 02) drove the frontend parameter form generation (Theme 03). This schema-first approach meant the frontend could render appropriate input widgets (number, string, enum, boolean, color picker) without hardcoding effect-specific UI, and validation occurs at both layers.

### E2E flaky test as systemic blocker (Themes 03–04)
The pre-existing flaky E2E test (BL-055, `project-creation.spec.ts`) blocked PR merges across two consecutive themes. This highlights the importance of fixing CI reliability issues before starting GUI-heavy development cycles.

### Feature composition through clean store interfaces (Theme 03)
The four GUI features were designed as standalone components with independent Zustand stores. This enabled smooth composition in the final workflow feature (004) without rework — catalog, form, and preview components plugged together cleanly.

### Documentation benefits from proximity to implementation (Themes 02, 04)
Architecture documentation updates were most efficient when done immediately after implementation (Theme 02 feature 003, Theme 04 feature 002), while context was fresh and divergence between design and reality was minimal.

## Technical Debt Summary

### High Priority

1. **Pre-existing flaky E2E test (BL-055)** — `gui/e2e/project-creation.spec.ts:31` modal hide assertion times out in CI because `POST /api/v1/projects` fails. Blocks PR merges for all GUI-touching features. Flagged across Themes 03 and 04.

2. **SPA fallback missing** — GUI routes (`/gui/effects`, `/gui/library`, etc.) return 404 when accessed directly via URL. Users bookmarking or refreshing pages get errors. E2E tests must navigate from root. Documented as known limitation in API specification.

### Medium Priority

3. **Large definitions.py module** — All 9 effect build functions and JSON schemas in a single file (~576 lines added in Theme 02). Should be split into domain modules (audio, video, transition) before adding more effects.

4. **EffectsPage orchestrator complexity** — `gui/src/pages/EffectsPage.tsx` orchestrates catalog, form, preview, clip selector, and effect stack. Consider extracting an `useEffectWorkflow` hook or splitting into sub-route pages before adding more workflow steps.

5. **Manual enum stub maintenance** — TransitionType and FadeCurve require hand-maintained entries in both `_core.pyi` stub files. Adding a third enum type should trigger investment in automated enum stub generation.

### Low Priority

6. **Stub file duplication** — Stubs exist in both `src/stoat_ferret_core/_core.pyi` and `stubs/stoat_ferret_core/_core.pyi`, both maintained manually.

7. **Transition storage as JSON column** — `transitions_json` stores an array of dicts as JSON in SQLite. Works at current scale but would need normalization for complex queries.

8. **Preview thumbnail not implemented (M2.9)** — M2.9 sub-item 5 (visual preview thumbnail) is placeholder only. Will require a backend render/thumbnail endpoint.

## Process Improvements

1. **Validate requirements against upstream sources during design** — Theme 01 discovered 59 actual FFmpeg xfade variants vs. 64 specified in requirements. Cross-checking numeric claims against upstream source documentation during design would avoid spec-vs-reality mismatches.

2. **Fix CI blockers before GUI development cycles** — Two themes were impacted by the same flaky E2E test. Establishing a CI health gate before starting GUI-heavy versions would prevent repeated friction.

3. **Consider domain-module splitting thresholds** — The `definitions.py` growth pattern suggests a rule: when a single module exceeds ~500 lines or serves 3+ effect domains, split into domain modules.

## Statistics

| Metric | Value |
|--------|-------|
| Themes completed | 4/4 |
| Features completed | 11/11 |
| Acceptance criteria passed | 131/131 |
| PRs merged | 6+ (#87–#92) |
| Commits | 7+ |
| Rust builders added | 7 (AmixBuilder, VolumeBuilder, AfadeBuilder, DuckingPattern, FadeBuilder, XfadeBuilder, AcrossfadeBuilder) |
| New API endpoints | 5 (POST /effects/transition, POST /effects/preview, PATCH /effects/{index}, DELETE /effects/{index}, GET /effects with schema) |
| New GUI components | 5 (EffectCatalog, EffectParameterForm, FilterPreview, ClipSelector, EffectStack) |
| New Zustand stores | 4 |
| Rust unit tests added | 89 |
| Python parity tests added | 88 |
| Backend tests added (registry/API) | 45+ |
| Frontend vitest tests added | 52 (101 → 143 total) |
| E2E Playwright tests added | 8 |
| Total pytest suite | 864 passed, 20 skipped |
| Total vitest suite | 143 tests |
| Coverage | ~91–92% |
| New runtime dependency | jsonschema |
| Roadmap milestones completed | M2.4, M2.5, M2.6, M2.8, M2.9 (4/5) |
