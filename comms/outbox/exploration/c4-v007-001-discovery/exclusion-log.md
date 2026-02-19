# Exclusion Log - C4 Discovery v007

All directories encountered during source tree traversal with inclusion/exclusion status and reason.

## Included Source Directories (37)

| Directory | Status | Reason |
|---|---|---|
| gui/e2e | included | Contains .spec.ts files |
| gui/src | included | Contains .tsx files |
| gui/src/components | included | Contains .tsx files |
| gui/src/components/__tests__ | included | Contains .test.tsx files |
| gui/src/hooks | included | Contains .ts files |
| gui/src/hooks/__tests__ | included | Contains .test.ts files |
| gui/src/pages | included | Contains .tsx files |
| gui/src/stores | included | Contains .ts files |
| rust/stoat_ferret_core/src | included | Contains .rs files |
| rust/stoat_ferret_core/src/bin | included | Contains .rs files |
| rust/stoat_ferret_core/src/clip | included | Contains .rs files |
| rust/stoat_ferret_core/src/ffmpeg | included | Contains .rs files |
| rust/stoat_ferret_core/src/sanitize | included | Contains .rs files |
| rust/stoat_ferret_core/src/timeline | included | Contains .rs files |
| scripts | included | Contains .py files |
| src/stoat_ferret | included | Contains .py files |
| src/stoat_ferret/api | included | Contains .py files |
| src/stoat_ferret/api/middleware | included | Contains .py files |
| src/stoat_ferret/api/routers | included | Contains .py files |
| src/stoat_ferret/api/schemas | included | Contains .py files |
| src/stoat_ferret/api/services | included | Contains .py files |
| src/stoat_ferret/api/websocket | included | Contains .py files |
| src/stoat_ferret/db | included | Contains .py files |
| src/stoat_ferret/effects | included | Contains .py files |
| src/stoat_ferret/ffmpeg | included | Contains .py files |
| src/stoat_ferret/jobs | included | Contains .py files |
| src/stoat_ferret_core | included | Contains .py and .pyi files |
| stubs/stoat_ferret_core | included | Contains .pyi files |
| tests | included | Contains .py files |
| tests/examples | included | Contains .py files |
| tests/test_api | included | Contains .py files |
| tests/test_blackbox | included | Contains .py files |
| tests/test_contract | included | Contains .py files |
| tests/test_coverage | included | Contains .py files |
| tests/test_doubles | included | Contains .py files |
| tests/test_jobs | included | Contains .py files |
| tests/test_security | included | Contains .py files |

## Excluded Directories

| Directory/Pattern | Reason |
|---|---|
| node_modules/ | Dependency tree (npm) |
| .git/ | Git internals |
| __pycache__/ | Python bytecode cache |
| .mypy_cache/ | Mypy type-checking cache |
| .pytest_cache/ | Pytest cache |
| dist/ | Build output |
| build/ | Build output |
| *.egg-info/ | Python package metadata |
| target/ | Rust/Cargo build output |
| .venv/, venv/, env/, .env/ | Virtual environments |
| .tox/ | Tox testing environments |
| .eggs/ | Python egg builds |
| docs/C4-Documentation/ | Existing C4 docs (not source) |
| docs/auto-dev/ | Auto-dev process docs |
| docs/ideation/ | Ideation documents |
| docs/design/ | Design documents (markdown only) |
| comms/ | MCP communication folders |
| gui/public/ | Static assets only |
| .github/ | CI/CD configuration (YAML only) |

## Delta Mode: Processing Decisions

| Directory | Decision | Reason |
|---|---|---|
| gui/e2e | PROCESS | Files changed: accessibility.spec.ts, effect-workshop.spec.ts, project-creation.spec.ts |
| gui/src | PROCESS | Files changed: App.tsx |
| gui/src/components | PROCESS | Files added: ClipSelector, EffectCatalog, EffectParameterForm, EffectStack, FilterPreview; modified: Navigation |
| gui/src/components/__tests__ | PROCESS | Files added: 5 new test files for effect components |
| gui/src/hooks | PROCESS | Files added: useEffectPreview.ts, useEffects.ts |
| gui/src/hooks/__tests__ | PROCESS | Files added: useEffectPreview.test.ts, useEffects.test.ts |
| gui/src/pages | PROCESS | Files added: EffectsPage.tsx |
| gui/src/stores | PROCESS | Files added: effectCatalogStore, effectFormStore, effectPreviewStore, effectStackStore |
| rust/stoat_ferret_core/src | PROCESS | Files changed: lib.rs |
| rust/stoat_ferret_core/src/bin | SKIP | No changes since v006 |
| rust/stoat_ferret_core/src/clip | SKIP | No changes since v006 |
| rust/stoat_ferret_core/src/ffmpeg | PROCESS | Files added: audio.rs, transitions.rs; modified: mod.rs |
| rust/stoat_ferret_core/src/sanitize | SKIP | No changes since v006 |
| rust/stoat_ferret_core/src/timeline | SKIP | No changes since v006 |
| scripts | SKIP | No changes since v006 |
| src/stoat_ferret | PROCESS | Ancestor of changed directories |
| src/stoat_ferret/api | PROCESS | Ancestor of changed directories |
| src/stoat_ferret/api/middleware | SKIP | No changes since v006 |
| src/stoat_ferret/api/routers | PROCESS | Files changed: effects.py |
| src/stoat_ferret/api/schemas | PROCESS | Files changed: effect.py |
| src/stoat_ferret/api/services | SKIP | No changes since v006 |
| src/stoat_ferret/api/websocket | SKIP | No changes since v006 |
| src/stoat_ferret/db | PROCESS | Files changed: models.py, project_repository.py, schema.py |
| src/stoat_ferret/effects | PROCESS | Files changed: definitions.py, registry.py |
| src/stoat_ferret/ffmpeg | SKIP | No changes since v006 |
| src/stoat_ferret/jobs | SKIP | No changes since v006 |
| src/stoat_ferret_core | PROCESS | Files changed: __init__.py, _core.pyi |
| stubs/stoat_ferret_core | PROCESS | Files changed: _core.pyi |
| tests | PROCESS | Files added: test_audio_builders.py, test_transition_builders.py |
| tests/examples | SKIP | No changes since v006 |
| tests/test_api | PROCESS | Files changed: test_effects.py |
| tests/test_blackbox | SKIP | No changes since v006 |
| tests/test_contract | PROCESS | Files changed: test_repository_parity.py |
| tests/test_coverage | SKIP | No changes since v006 |
| tests/test_doubles | SKIP | No changes since v006 |
| tests/test_jobs | SKIP | No changes since v006 |
| tests/test_security | SKIP | No changes since v006 |
