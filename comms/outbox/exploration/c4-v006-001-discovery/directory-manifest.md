# C4 Directory Manifest

## Metadata
- Mode: delta
- Total Directories: 36
- Directories to Process: 14
- Batch Count: 2
- Previous Version: v005
- Previous C4 Commit: 9fe4727c11a0e7d0ac25bf97448b7b708ac7d837
- Generated: 2026-02-19 UTC

## Batch 1
- rust/stoat_ferret_core/src
- rust/stoat_ferret_core/src/ffmpeg
- src/stoat_ferret/effects
- src/stoat_ferret/api
- src/stoat_ferret/api/routers
- src/stoat_ferret/api/schemas
- src/stoat_ferret/db

## Batch 2
- src/stoat_ferret_core
- gui/src/components
- tests
- tests/test_api
- tests/test_blackbox
- tests/test_contract
- tests/test_coverage

## Unchanged Directories (delta mode)
- rust/stoat_ferret_core/src/bin (existing: c4-code-rust-stoat-ferret-core-bin.md)
- rust/stoat_ferret_core/src/clip (existing: c4-code-rust-stoat-ferret-core-clip.md)
- rust/stoat_ferret_core/src/timeline (existing: c4-code-rust-stoat-ferret-core-timeline.md)
- rust/stoat_ferret_core/src/sanitize (existing: c4-code-rust-stoat-ferret-core-sanitize.md)
- src/stoat_ferret (existing: c4-code-stoat-ferret.md)
- src/stoat_ferret/api/middleware (existing: c4-code-stoat-ferret-api-middleware.md)
- src/stoat_ferret/api/services (existing: c4-code-stoat-ferret-api-services.md)
- src/stoat_ferret/api/websocket (existing: c4-code-stoat-ferret-api-websocket.md)
- src/stoat_ferret/ffmpeg (existing: c4-code-stoat-ferret-ffmpeg.md)
- src/stoat_ferret/jobs (existing: c4-code-stoat-ferret-jobs.md)
- gui/src (existing: c4-code-gui-src.md)
- gui/src/hooks (existing: c4-code-gui-hooks.md)
- gui/src/pages (existing: c4-code-gui-pages.md)
- gui/src/stores (existing: c4-code-gui-stores.md)
- gui/src/components/__tests__ (existing: c4-code-gui-components-tests.md)
- gui/src/hooks/__tests__ (existing: c4-code-gui-hooks-tests.md)
- scripts (existing: c4-code-scripts.md)
- tests/test_doubles (existing: c4-code-tests-test-doubles.md)
- tests/test_jobs (existing: c4-code-tests-test-jobs.md)
- tests/test_security (existing: c4-code-tests-test-security.md)
- tests/examples (existing: c4-code-tests-examples.md)

## Changed Files (delta mode)
- gui/src/components/ProjectCard.tsx (modified)
- rust/stoat_ferret_core/src/ffmpeg/drawtext.rs (added)
- rust/stoat_ferret_core/src/ffmpeg/expression.rs (added)
- rust/stoat_ferret_core/src/ffmpeg/filter.rs (modified)
- rust/stoat_ferret_core/src/ffmpeg/mod.rs (modified)
- rust/stoat_ferret_core/src/ffmpeg/speed.rs (added)
- rust/stoat_ferret_core/src/lib.rs (modified)
- src/stoat_ferret/api/app.py (modified)
- src/stoat_ferret/api/routers/effects.py (added)
- src/stoat_ferret/api/schemas/clip.py (modified)
- src/stoat_ferret/api/schemas/effect.py (added)
- src/stoat_ferret/db/clip_repository.py (added)
- src/stoat_ferret/db/models.py (modified)
- src/stoat_ferret/db/schema.py (modified)
- src/stoat_ferret/effects/__init__.py (added)
- src/stoat_ferret/effects/definitions.py (added)
- src/stoat_ferret/effects/registry.py (added)
- src/stoat_ferret_core/__init__.py (modified)
- src/stoat_ferret_core/_core.pyi (modified)
- tests/test_api/test_effects.py (added)
- tests/test_blackbox/test_core_workflow.py (modified)
- tests/test_clip_model.py (modified)
- tests/test_contract/test_ffmpeg_contract.py (modified)
- tests/test_coverage/test_import_fallback.py (modified)
- tests/test_db_schema.py (modified)
- tests/test_pyo3_bindings.py (modified)
