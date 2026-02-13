# Risk Assessment: v006 — Effects Engine Foundation

## Risk: Clip effects storage model design is unresolved

- **Original severity**: high
- **Category**: Investigate now
- **Investigation**: Queried the codebase for the current clip model across all three layers (Pydantic schema, DB repository, Rust Clip type) and existing patterns for storing complex/JSON data.
- **Finding**: The clip model has no `effects` field in any layer. However, the project already uses a JSON TEXT pattern for complex data — the audit log stores `changes_json TEXT` in SQLite, serialized via `json.dumps()`. The Pydantic schema (`src/stoat_ferret/api/schemas/clip.py`), Python dataclass (`src/stoat_ferret/db/models.py`), DB schema (`src/stoat_ferret/db/schema.py`), and both repository implementations (`AsyncSQLiteClipRepository`, `AsyncInMemoryClipRepository`) all follow consistent patterns. The Rust `Clip` type (`rust/stoat_ferret_core/src/clip/mod.rs`) holds only source/position/duration fields — it does not need an effects field because Rust handles filter generation, not effect storage.
- **Resolution**: Effects stored as `effects_json TEXT` column in SQLite (matching audit log pattern). Python Clip dataclass gets an `effects` field (JSON string). Pydantic schemas get typed `effects` list. Rust `Clip` type unchanged — Python owns effect storage, Rust owns filter generation (LRN-011 boundary). Alembic migration adds the column. This confirms Task 005's assumption and validates the Feature 002 (clip-effect-model) design.
- **Affected themes/features**: Theme 03 Feature 002 (clip-effect-model), Theme 03 Feature 003 (text-overlay-apply)

## Risk: Effect registry pattern needs upfront design for extensibility

- **Original severity**: high
- **Category**: Investigate now
- **Investigation**: Queried the codebase for the `create_app()` DI pattern, all existing services, dependency access patterns from route handlers, and any existing registry or discovery patterns.
- **Finding**: The project has a well-established DI pattern: `create_app()` accepts keyword-only optional params, stores on `app.state`, and route handlers access via `Depends()` functions that check `app.state` with production fallback. The job queue (`AsyncioJobQueue`) already implements a dictionary-based registry pattern: `register_handler(job_type, handler)` with handlers stored in `self._handlers: dict[str, JobHandler]` and registered during app lifespan. The `EventType` enum provides an enumeration pattern for known types. Six existing services are injected (video/project/clip repositories, job queue, ws_manager, thumbnail service).
- **Resolution**: The EffectRegistry follows the established job handler registry pattern: dictionary-based registration, injected via `create_app()` kwargs, stored on `app.state.effect_registry`, accessed via `Depends()` function. Each effect provides a registration object with name, description, parameter Pydantic model (auto JSON Schema via `.model_json_schema()`), and AI hints dict. Effects registered during app lifespan (like job handlers at line 74 of app.py). This resolves the extensibility concern — v007 effects simply call `registry.register()`.
- **Affected themes/features**: Theme 03 Feature 001 (effect-discovery), Theme 03 Feature 003 (text-overlay-apply)

## Risk: Task 004 research was incomplete

- **Original severity**: medium
- **Category**: Accept with mitigation
- **Investigation**: The research gaps (FFmpeg expression functions, graph validation rules, drawtext parameters, speed control approach) were partially addressed by investigations in this task. DeepWiki confirmed FFmpeg expression function availability and setpts behavior.
- **Finding**: The critical research questions have been answered: (1) FFmpeg expressions support `between()`, `if()`, `min()`, `max()`, `gte()`, `lte()`, `t`, `PTS`, `n` — all functions needed for v006. (2) `setpts=PTS/factor` is frame-rate independent. (3) Existing codebase has `escape_filter_text()` and `validate_speed()` already in the Rust sanitize module. Remaining gaps (detailed drawtext parameter priority, graph validation edge cases) are standard FFmpeg documentation lookups addressable during implementation.
- **Resolution**: No design changes needed. Mitigation: implementation workers consult FFmpeg documentation during feature implementation. The expression function whitelist for BL-037 is now defined (see Risk 4 below). Accept remaining gaps as implementation-time lookups.
- **Affected themes/features**: All themes (minor impact)

## Risk: FFmpeg filter expression subset scope creep

- **Original severity**: medium
- **Category**: Investigate now
- **Investigation**: Researched FFmpeg expression functions via DeepWiki, cross-referenced with BL-040 (drawtext) and BL-041 (speed control) requirements, and reviewed existing Rust sanitize module.
- **Finding**: v006 downstream consumers need a specific, bounded set of expression functions: (1) BL-040 drawtext alpha animation needs: `between(t, start, end)`, `if(x, y, z)`, `min(x, y)`, `max(x, y)`, time variable `t`, and arithmetic operators. (2) BL-041 speed control needs only `PTS/factor` — simple arithmetic, no expression engine functions. (3) `enable` expressions use `between()`. (4) FFmpeg also provides `gte()`, `lte()`, `eq()`, `gt()`, `lt()` for comparisons — useful but not required for v006.
- **Resolution**: Define a v006 expression function whitelist: **Required**: `between`, `if`, `min`, `max` + arithmetic operators (`+`, `-`, `*`, `/`) + time variables (`t`, `PTS`, `n`). **Optional** (include if trivial): `gte`, `lte`, `eq`, `gt`, `lt`. The expression engine's extensibility (add new functions without breaking API) handles v007 needs. This bounds the scope and prevents creep.
- **Affected themes/features**: Theme 01 Feature 001 (expression-engine), Theme 02 Feature 002 (drawtext-builder)

## Risk: Rust coverage threshold gap

- **Original severity**: medium
- **Category**: Accept with mitigation
- **Investigation**: N/A — this is a tracking concern, not a resolvable design question.
- **Finding**: Current Rust coverage is ~75% against 90% target. v006 adds 5 significant Rust features (expression engine, graph validation, composition, drawtext, speed control) with comprehensive test requirements (proptest, edge cases, contract tests). Whether new module coverage closes the overall gap depends on existing code gaps outside v006's scope.
- **Resolution**: Mitigation: (1) All v006 Rust features must achieve >90% module-level coverage via proptest and unit tests. (2) Track overall crate coverage after Theme 02 completes. (3) If overall coverage still below 90%, add a quality-gaps.md entry for targeted coverage improvement (per retrospective insight). No design change needed.
- **Affected themes/features**: Theme 01 (all features), Theme 02 (all features)

## Risk: BL-043 depends on two substantial impacts being resolved

- **Original severity**: medium
- **Category**: Accept with mitigation
- **Investigation**: Reviewed the Theme 03 feature sequencing and DI pattern investigation results.
- **Finding**: The design already mitigates this by splitting BL-043 into two features (model + endpoint) and sequencing discovery first. The `create_app()` DI pattern is well-established with 6 existing services — adding the effect registry follows the same pattern and should not be a bottleneck. The clip model extension follows the audit log JSON pattern — also well-established. Both are "substantial" in scope but use proven patterns.
- **Resolution**: No design change needed. The existing 3-feature split in Theme 03 (discovery → model → endpoint) provides sufficient structure. Both substantial impacts use established codebase patterns. Monitor Theme 03 Feature 001 and 002 completion times — if Feature 001 takes >50% of Theme 03's time budget, raise a flag.
- **Affected themes/features**: Theme 03 (all features)

## Risk: Contract tests require FFmpeg binary in CI

- **Original severity**: low
- **Category**: Investigate now
- **Investigation**: Queried CI workflow, existing contract test infrastructure, FFmpegExecutor implementations, and test markers.
- **Finding**: **Fully resolved.** CI installs FFmpeg on all 9 matrix entries (3 OS x 3 Python) via `AnimMouse/setup-ffmpeg@v1` (ci.yml line 55-56). The project has a mature 3-tier executor system: `RealFFmpegExecutor` (subprocess), `RecordingFFmpegExecutor` (captures to JSON), `FakeFFmpegExecutor` (replays from JSON, with strict mode). 21 existing contract tests — 6 require real FFmpeg (skip gracefully via `@requires_ffmpeg` marker), 15 use mocks. The record-replay pattern (LRN-008) and single-matrix pattern (LRN-015) are both implemented and working.
- **Resolution**: No design change needed. BL-040 contract tests follow the existing pattern: use `RecordingFFmpegExecutor` to capture real FFmpeg output, `FakeFFmpegExecutor` for replay in CI. The `@pytest.mark.contract` and `@requires_ffmpeg` markers are already registered. Filter-level validation (`ffmpeg -filter_complex`) uses the same `RealFFmpegExecutor.run()` with different args.
- **Affected themes/features**: Theme 02 Feature 002 (drawtext-builder)

## Risk: Drop-frame timecode may intersect with speed control

- **Original severity**: low
- **Category**: Investigate now
- **Investigation**: Researched FFmpeg setpts filter behavior via DeepWiki, specifically whether PTS manipulation is frame-rate dependent and whether drop-frame timecode affects PTS calculations.
- **Finding**: **Fully resolved.** DeepWiki confirms: (1) `setpts` operates on PTS values directly via the `filter_frame` → `eval_pts` → `av_expr_eval` pipeline in `libavfilter/setpts.c`. (2) PTS values are timestamps in the stream's timebase, independent of frame rate. (3) Drop-frame timecode is a display convention for NTSC video — it affects how frame numbers map to timecode strings but does not affect the underlying linear PTS values. (4) `setpts=PTS/2.0` halves PTS for every frame regardless of frame rate (23.976, 29.97, 30, 60 fps) or drop-frame timecode.
- **Resolution**: No design change needed. Task 005's assumption confirmed: speed control via `PTS/factor` is frame-rate independent. Drop-frame timecode is irrelevant to PTS calculations. BL-041 implementation can proceed without special frame-rate handling.
- **Affected themes/features**: Theme 02 Feature 003 (speed-control)
