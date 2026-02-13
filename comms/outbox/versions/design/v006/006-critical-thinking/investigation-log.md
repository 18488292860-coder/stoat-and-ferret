# Investigation Log: v006 — Critical Thinking

## Investigation 1: Clip Effects Storage Model

**Method:** Codebase exploration — queried Python schemas, DB models, repository implementations, Rust types, and existing JSON storage patterns.

**Queries and Results:**

1. **Pydantic clip schemas** (`src/stoat_ferret/api/schemas/clip.py`): `ClipCreate`, `ClipUpdate`, `ClipResponse` — no effects field. Response has: id, project_id, source_video_id, in_point, out_point, timeline_position, created_at, updated_at.

2. **Python Clip dataclass** (`src/stoat_ferret/db/models.py:59-109`): Same fields as schema plus `validate()` method that delegates to Rust. Uses `from stoat_ferret_core import Clip as RustClip, validate_clip`.

3. **DB schema** (`src/stoat_ferret/db/schema.py:96-115`): SQLite `clips` table with TEXT/INTEGER columns. No JSON fields. Alembic migration exists (`alembic/versions/39896ab3d0b7_add_clips_table.py`).

4. **Repository implementations** (`src/stoat_ferret/db/clip_repository.py`): Protocol (`AsyncClipRepository`) with `add/get/list_by_project/update/delete`. SQLite implementation uses `_row_to_clip()` for deserialization. In-memory implementation uses `copy.deepcopy()`.

5. **Rust Clip** (`rust/stoat_ferret_core/src/clip/mod.rs:63-139`): `source_path: String`, `in_point: Position`, `out_point: Position`, `source_duration: Option<Duration>`. No effects field.

6. **JSON storage precedent** (`src/stoat_ferret/db/audit.py` + `schema.py:68-77`): Audit log stores `changes_json TEXT` — serialized via `json.dumps()`, stored as TEXT in SQLite.

**Conclusion:** Effects can follow the audit log JSON TEXT pattern. Rust Clip unchanged (LRN-011: Rust=input safety, Python=business logic). Alembic migration adds `effects_json TEXT` column.

---

## Investigation 2: Effect Registry DI Pattern

**Method:** Codebase exploration — queried `create_app()`, all DI services, route handler dependency access, existing registry patterns.

**Queries and Results:**

1. **`create_app()`** (`src/stoat_ferret/api/app.py:90-162`): Accepts keyword-only optional params: `video_repository`, `project_repository`, `clip_repository`, `job_queue`, `ws_manager`, `gui_static_path`. Stores on `app.state`. Sets `_deps_injected=True` flag.

2. **Lifespan** (`app.py:37-87`): Checks `_deps_injected` flag — if set, skips DB setup (test mode). If not, creates production services including DB connection, job queue, repositories.

3. **Route dependency pattern** (e.g., `videos.py:34-49`): `get_repository(request)` checks `request.app.state.video_repository` — if found, returns it; otherwise creates production implementation from `request.app.state.db`.

4. **Job handler registry** (`src/stoat_ferret/jobs/queue.py`): `AsyncioJobQueue.register_handler(job_type, handler)` stores handlers in `self._handlers: dict[str, JobHandler]`. Registration happens in lifespan (`app.py:74`).

5. **Event enumeration** (`src/stoat_ferret/api/websocket/events.py:12-19`): `EventType(str, Enum)` with `HEALTH_STATUS`, `SCAN_STARTED`, etc.

6. **Existing services count**: 6 services injected (3 repositories, job queue, ws_manager, thumbnail service).

**Conclusion:** EffectRegistry follows the job handler registry dictionary pattern, injected via `create_app()` kwargs. Test DI pattern confirmed — tests do NOT use `dependency_overrides` but direct `app.state` injection (`tests/test_api/conftest.py`).

---

## Investigation 3: CI FFmpeg Availability

**Method:** Codebase exploration — queried CI workflow files, contract test infrastructure, FFmpeg executor implementations.

**Queries and Results:**

1. **CI workflow** (`.github/workflows/ci.yml`): FFmpeg installed via `AnimMouse/setup-ffmpeg@v1` (line 55-56) on ALL 9 matrix entries (ubuntu/windows/macos x Python 3.10/3.11/3.12).

2. **Contract tests** (`tests/test_contract/test_ffmpeg_contract.py`): 21 tests in 4 classes. Stage 2 (record-replay with mocks), Stage 3 (real executor — `@requires_ffmpeg`), Stage 4 (strict mode). 6 tests need real FFmpeg, 15 use mocks.

3. **Executor implementations** (`src/stoat_ferret/ffmpeg/executor.py`):
   - `RealFFmpegExecutor`: subprocess execution
   - `RecordingFFmpegExecutor`: wraps another executor, records to JSON
   - `FakeFFmpegExecutor`: replays from JSON, optional strict mode

4. **FFmpeg detection** (`tests/conftest.py:14-26`): `shutil.which("ffmpeg")` check, `@requires_ffmpeg` skip marker.

5. **Test markers** (`pyproject.toml:64-71`): `contract`, `requires_ffmpeg`, `property` markers registered.

**Conclusion:** CI FFmpeg support fully operational. Contract test infrastructure extensible to filter-level validation. No changes needed.

---

## Investigation 4: FFmpeg Expression Function Whitelist

**Method:** Codebase exploration + DeepWiki research on FFmpeg expression evaluation system.

**Queries and Results:**

1. **Existing Rust filter module** (`rust/stoat_ferret_core/src/ffmpeg/filter.rs`): `Filter`, `FilterChain`, `FilterGraph` types with helpers (`scale`, `pad`, `format`, `concat`). No expression engine — parameters are key=value strings only.

2. **Existing sanitize module** (`rust/stoat_ferret_core/src/sanitize/mod.rs`): `escape_filter_text()`, `validate_speed()` (0.25-4.0 range), `validate_volume()`, `validate_crf()`. Whitelist validation pattern already established (LRN-003).

3. **DeepWiki: FFmpeg expression functions** — Confirmed available functions:
   - Conditional: `if(x, y)`, `if(x, y, z)`
   - Range: `between(x, min, max)`
   - Math: `min(x, y)`, `max(x, y)`
   - Comparison: `gte(x, y)`, `lte(x, y)`, `eq(x, y)`, `gt(x, y)`, `lt(x, y)`
   - Arithmetic: `+`, `-`, `*`, `/`
   - Variables: `t` (timestamp seconds), `PTS` (original PTS), `n` (frame number), `w`, `h`, `TB`, `FRAME_RATE`

4. **Downstream requirements analysis:**
   - BL-040 drawtext alpha: needs `between(t, start, end)`, `if(x, y, z)`, `min(x, y)`, `t`, arithmetic
   - BL-041 speed: needs only `PTS/factor` — simple arithmetic, no functions
   - BL-040 enable: needs `between(t, start, end)`

**Conclusion:** v006 whitelist = `{between, if, min, max}` + arithmetic + `{t, PTS, n}`. Optional: `{gte, lte, eq, gt, lt}`. Extensible for v007.

---

## Investigation 5: Drop-Frame Timecode vs Speed Control

**Method:** DeepWiki research on FFmpeg setpts filter internals.

**Queries and Results:**

1. **DeepWiki: setpts implementation** — Filter implemented in `libavfilter/setpts.c`. The `filter_frame` function calls `eval_pts` which sets `PTS` variable to original frame PTS (converted to double via `TS2D`), evaluates expression via `av_expr_eval`, converts result back to `int64_t` via `D2TS`.

2. **Frame-rate independence** — PTS values are timestamps in the stream's timebase, independent of frame rate. `FRAME_RATE` variable is available but `setpts=PTS/2.0` does not use it.

3. **Drop-frame timecode** — A display convention for NTSC video (29.97 fps). Affects frame-number-to-timecode mapping but not underlying PTS values. PTS values remain linear and continuous regardless of drop-frame flags.

**Conclusion:** `setpts=PTS/factor` is frame-rate independent. Drop-frame timecode does not affect PTS calculations. Task 005's assumption confirmed with evidence.
