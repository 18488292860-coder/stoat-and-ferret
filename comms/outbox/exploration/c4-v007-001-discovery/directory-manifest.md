# C4 Directory Manifest

## Metadata
- Mode: delta
- Total Directories: 37
- Directories to Process: 21
- Batch Count: 2
- Previous Version: v006
- Previous C4 Commit: 97398aa32c3a3349535264d47982fd1c348ad5df
- Generated: 2026-02-19 21:31 UTC

## Batch 1
- gui/e2e
- gui/src
- gui/src/components
- gui/src/components/__tests__
- gui/src/hooks
- gui/src/hooks/__tests__
- gui/src/pages
- gui/src/stores
- rust/stoat_ferret_core/src
- rust/stoat_ferret_core/src/ffmpeg

## Batch 2
- src/stoat_ferret
- src/stoat_ferret/api
- src/stoat_ferret/api/routers
- src/stoat_ferret/api/schemas
- src/stoat_ferret/db
- src/stoat_ferret/effects
- src/stoat_ferret_core
- stubs/stoat_ferret_core
- tests
- tests/test_api
- tests/test_contract

## Unchanged Directories
- rust/stoat_ferret_core/src/bin (existing: c4-code-rust-stoat-ferret-core-bin.md)
- rust/stoat_ferret_core/src/clip (existing: c4-code-rust-stoat-ferret-core-clip.md)
- rust/stoat_ferret_core/src/sanitize (existing: c4-code-rust-stoat-ferret-core-sanitize.md)
- rust/stoat_ferret_core/src/timeline (existing: c4-code-rust-stoat-ferret-core-timeline.md)
- scripts (existing: c4-code-scripts.md)
- src/stoat_ferret/api/middleware (existing: c4-code-stoat-ferret-api-middleware.md)
- src/stoat_ferret/api/services (existing: c4-code-stoat-ferret-api-services.md)
- src/stoat_ferret/api/websocket (existing: c4-code-stoat-ferret-api-websocket.md)
- src/stoat_ferret/ffmpeg (existing: c4-code-stoat-ferret-ffmpeg.md)
- src/stoat_ferret/jobs (existing: c4-code-stoat-ferret-jobs.md)
- tests/examples (existing: c4-code-tests-examples.md)
- tests/test_blackbox (existing: c4-code-tests-test-blackbox.md)
- tests/test_coverage (existing: c4-code-tests-test-coverage.md)
- tests/test_doubles (existing: c4-code-tests-test-doubles.md)
- tests/test_jobs (existing: c4-code-tests-test-jobs.md)
- tests/test_security (existing: c4-code-tests-test-security.md)

## Changed Files
- gui/e2e/accessibility.spec.ts (modified)
- gui/e2e/effect-workshop.spec.ts (added)
- gui/e2e/project-creation.spec.ts (modified)
- gui/src/App.tsx (modified)
- gui/src/components/ClipSelector.tsx (added)
- gui/src/components/EffectCatalog.tsx (added)
- gui/src/components/EffectParameterForm.tsx (added)
- gui/src/components/EffectStack.tsx (added)
- gui/src/components/FilterPreview.tsx (added)
- gui/src/components/Navigation.tsx (modified)
- gui/src/components/__tests__/ClipSelector.test.tsx (added)
- gui/src/components/__tests__/EffectCatalog.test.tsx (added)
- gui/src/components/__tests__/EffectParameterForm.test.tsx (added)
- gui/src/components/__tests__/EffectStack.test.tsx (added)
- gui/src/components/__tests__/FilterPreview.test.tsx (added)
- gui/src/hooks/__tests__/useEffectPreview.test.ts (added)
- gui/src/hooks/__tests__/useEffects.test.ts (added)
- gui/src/hooks/useEffectPreview.ts (added)
- gui/src/hooks/useEffects.ts (added)
- gui/src/pages/EffectsPage.tsx (added)
- gui/src/stores/effectCatalogStore.ts (added)
- gui/src/stores/effectFormStore.ts (added)
- gui/src/stores/effectPreviewStore.ts (added)
- gui/src/stores/effectStackStore.ts (added)
- rust/stoat_ferret_core/src/ffmpeg/audio.rs (added)
- rust/stoat_ferret_core/src/ffmpeg/mod.rs (modified)
- rust/stoat_ferret_core/src/ffmpeg/transitions.rs (added)
- rust/stoat_ferret_core/src/lib.rs (modified)
- src/stoat_ferret/api/routers/effects.py (modified)
- src/stoat_ferret/api/schemas/effect.py (modified)
- src/stoat_ferret/db/models.py (modified)
- src/stoat_ferret/db/project_repository.py (modified)
- src/stoat_ferret/db/schema.py (modified)
- src/stoat_ferret/effects/definitions.py (modified)
- src/stoat_ferret/effects/registry.py (modified)
- src/stoat_ferret_core/__init__.py (modified)
- src/stoat_ferret_core/_core.pyi (modified)
- stubs/stoat_ferret_core/_core.pyi (modified)
- tests/test_api/test_effects.py (modified)
- tests/test_audio_builders.py (added)
- tests/test_contract/test_repository_parity.py (modified)
- tests/test_transition_builders.py (added)
