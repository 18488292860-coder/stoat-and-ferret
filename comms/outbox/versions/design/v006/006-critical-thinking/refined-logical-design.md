# Refined Logical Design: v006 — Effects Engine Foundation

## Version Overview

**Version:** v006
**Description:** Effects Engine Foundation — Build the Rust filter expression engine, graph validation, filter composition, text overlay builder, speed control, and effect discovery/application API endpoints.

**Goals:**
- Establish a type-safe FFmpeg filter expression DSL in Rust (M2.1)
- Implement graph validation and composition system for complex filter chains (M2.1)
- Build drawtext and speed control filter builders (M2.2, M2.3)
- Bridge Rust effects engine to Python API with discovery and application endpoints

**Scope:** 7 backlog items (BL-037-BL-043), 3 themes, 9 features total.

**Changes from Task 005:** No structural changes to themes, features, or execution order. Risk investigations confirmed all assumptions. Design refinements are scoping clarifications, not structural modifications.

---

## Theme 01: `01-filter-expression-infrastructure`

**Goal:** Build the foundational Rust infrastructure — a type-safe expression engine and graph validation system.

**Backlog Items:** BL-037, BL-038

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-expression-engine` | Implement type-safe FFmpeg expression AST with builder API, serialization, and property-based tests | BL-037 | None |
| 2 | `002-graph-validation` | Add pad matching validation, unconnected pad detection, and cycle detection to FilterGraph | BL-038 | None |

### Feature 001-expression-engine (BL-037)

**Goal:** Implement a Rust expression AST covering enable, alpha, time, and arithmetic FFmpeg expressions with a fluent builder API (LRN-001: `PyRefMut` chaining), serialization to valid FFmpeg syntax, property-based tests via proptest, and PyO3 bindings with type stubs.

**Key design decisions:**
- Expression function names validated against whitelist of known FFmpeg functions (LRN-003)
- Builder uses `PyRefMut<'_, Self>` for method chaining (LRN-001)
- Rust owns expression construction and validation; Python consumes via bindings (LRN-011)

**Refinement from risk investigation — Expression function whitelist:**
- **Required functions:** `between`, `if`, `min`, `max`
- **Required operators:** `+`, `-`, `*`, `/` (arithmetic)
- **Required variables:** `t` (timestamp seconds), `PTS` (presentation timestamp), `n` (frame number)
- **Optional (include if trivial):** `gte`, `lte`, `eq`, `gt`, `lt` (comparison functions)
- **Extensibility:** New functions added via whitelist extension without breaking existing API
- **Evidence:** DeepWiki confirmed all functions available in FFmpeg's `AVExpr` system. `between` and `if` needed by BL-040 drawtext. `PTS` needed by BL-041 speed control. Comparison functions useful but not required for v006 consumers.

### Feature 002-graph-validation (BL-038)

**Goal:** Add validation to the existing `FilterGraph` type: pad label matching, unconnected pad detection, cycle detection via topological sort, and actionable error messages.

**Key design decisions:**
- Extends existing `FilterGraph` (`rust/stoat_ferret_core/src/ffmpeg/filter.rs`) rather than creating a new type
- Validation runs automatically before serialization
- Error messages include guidance on how to fix the graph (per AC4)

---

## Theme 02: `02-filter-builders-and-composition`

**Goal:** Build the concrete filter builders and composition system. These features consume Theme 01's infrastructure.

**Backlog Items:** BL-039, BL-040, BL-041

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-filter-composition` | Build chain/branch/merge composition API with automatic pad management and validation | BL-039 | Theme 01 Feature 002 (graph validation) |
| 2 | `002-drawtext-builder` | Implement drawtext filter builder with position, styling, alpha animation, and contract tests | BL-040 | Theme 01 Feature 001 (expression engine) |
| 3 | `003-speed-control` | Implement setpts/atempo builders with automatic atempo chaining for >2x speeds | BL-041 | None (within theme) |

### Feature 001-filter-composition (BL-039)

**Goal:** Implement chain, branch, and merge composition modes with automatic pad label management and automatic validation of composed graphs.

**Key design decisions:**
- Composition automatically manages pad labels (no manual label assignment)
- Composed graphs validated automatically via Theme 01's graph validation
- Builder API uses `PyRefMut` chaining pattern (LRN-001)

### Feature 002-drawtext-builder (BL-040)

**Goal:** Build a structured drawtext filter builder supporting absolute/centered/margin-based positioning, font/color/shadow/box styling, fade in/out alpha animation via expression engine, with contract tests.

**Key design decisions:**
- Alpha animation uses expression engine from Theme 01 Feature 001
- Specifically uses: `alpha='min(1, (t-start)/fade_duration)'` for fade-in, `between(t, start, end)` for enable
- Contract tests leverage record-replay pattern (LRN-008) using existing `RecordingFFmpegExecutor`/`FakeFFmpegExecutor` infrastructure
- Contract tests run on all CI matrix entries (FFmpeg available on all 9 via `AnimMouse/setup-ffmpeg@v1`)
- Existing `escape_filter_text()` in Rust sanitize module handles text escaping
- Builder pattern with PyO3 bindings and type stubs (incremental binding rule)

### Feature 003-speed-control (BL-041)

**Goal:** Implement setpts (video speed) and atempo (audio speed) builders with 0.25x-4.0x range, automatic atempo chaining for >2x factors, audio drop option, and validation.

**Key design decisions:**
- Atempo auto-chaining: factors >2x decomposed into chained atempo filters (FFmpeg limitation)
- Independent of expression engine (uses simple `PTS/factor` arithmetic)
- **Confirmed frame-rate independent:** setpts operates on PTS values directly via `libavfilter/setpts.c` `eval_pts` pipeline. Drop-frame timecode does not affect PTS calculations. Works correctly for all frame rates (23.976, 29.97, 30, 60 fps).
- Existing `validate_speed()` in Rust sanitize module handles range validation (0.25-4.0)

---

## Theme 03: `03-effects-api-layer`

**Goal:** Bridge the Rust effects engine to the Python API layer with discovery and application endpoints.

**Backlog Items:** BL-042, BL-043

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-effect-discovery` | Create GET /effects endpoint with registry service, parameter JSON schemas, and AI hints | BL-042 | Theme 02 Features 002, 003 |
| 2 | `002-clip-effect-model` | Extend clip data model with effects storage across Python schema, DB repository, and Rust type | BL-043 (partial) | Feature 001 |
| 3 | `003-text-overlay-apply` | Create POST endpoint to apply text overlay to clips with validation, persistence, and filter preview | BL-043 (partial) | Feature 002 |

### Feature 001-effect-discovery (BL-042)

**Goal:** Implement `GET /effects` returning available effects with name, description, parameter JSON schemas, and AI hints.

**Refinement from risk investigation — Registry design:**
- **Pattern:** Dictionary-based registry following `AsyncioJobQueue.register_handler()` pattern (`src/stoat_ferret/jobs/queue.py`)
- **DI:** `EffectRegistry` injected via `create_app()` kwargs, stored on `app.state.effect_registry`
- **Route access:** Via `Depends()` function checking `app.state` (same as `get_repository()` pattern in `videos.py:34-49`)
- **Registration:** Effects registered during app lifespan (like job handlers at `app.py:74`)
- **Schema generation:** Parameter Pydantic models use `.model_json_schema()` for auto JSON Schema
- **Extensibility:** v007 effects call `registry.register()` — no changes to registry infrastructure needed
- Server-side sorting for effect listing from the start (per retrospective insight)

### Feature 002-clip-effect-model (BL-043, partial)

**Goal:** Extend the clip data model to support an effects list.

**Refinement from risk investigation — Storage design:**
- **DB layer:** Add `effects_json TEXT` column to clips table via Alembic migration (follows audit log `changes_json TEXT` pattern from `src/stoat_ferret/db/schema.py:68-77`)
- **Python dataclass:** Add `effects` field to `Clip` dataclass (`src/stoat_ferret/db/models.py`) — JSON string, parsed via helper method
- **Pydantic schemas:** Add typed `effects` list to `ClipResponse` (`src/stoat_ferret/api/schemas/clip.py`)
- **Repository:** Update `_row_to_clip()` in both `AsyncSQLiteClipRepository` and `AsyncInMemoryClipRepository`
- **Rust Clip:** No changes — Python owns effect storage, Rust owns filter generation (LRN-011)
- Effect configuration format aligned with registry schema from Feature 001

### Feature 003-text-overlay-apply (BL-043, partial)

**Goal:** Implement POST endpoint to apply text overlay parameters to a clip, validate via Rust drawtext builder, store effect configuration in clip model, return generated FFmpeg filter string, and emit WebSocket events.

**Key design decisions:**
- Validation errors from Rust surface as structured API error responses
- Response includes FFmpeg filter string for transparency (architectural principle)
- WebSocket events (`effect.applied`, `effect.removed`) for real-time updates — extends existing `EventType` enum (`src/stoat_ferret/api/websocket/events.py`)
- Black box test covers apply -> verify filter string flow (AC5)

---

## Execution Order

### Theme Dependencies

```
Theme 01 (filter-expression-infrastructure)
    |- Feature 001-expression-engine --> Theme 02 Feature 002 (drawtext)
    +- Feature 002-graph-validation --> Theme 02 Feature 001 (composition)

Theme 02 (filter-builders-and-composition)
    |- Feature 002-drawtext-builder --> Theme 03 Feature 001 (discovery)
    +- Feature 003-speed-control ----> Theme 03 Feature 001 (discovery)

Theme 03 (effects-api-layer)
    |- Feature 001-effect-discovery
    |- Feature 002-clip-effect-model (depends on Feature 001)
    +- Feature 003-text-overlay-apply (depends on Feature 002)
```

### Execution Sequence

1. **Theme 01, Feature 001**: expression-engine (BL-037) — no dependencies
2. **Theme 01, Feature 002**: graph-validation (BL-038) — no dependencies (parallel with F001)
3. **Theme 02, Feature 001**: filter-composition (BL-039) — needs Theme 01 F002
4. **Theme 02, Feature 002**: drawtext-builder (BL-040) — needs Theme 01 F001
5. **Theme 02, Feature 003**: speed-control (BL-041) — independent, sequenced after F002
6. **Theme 03, Feature 001**: effect-discovery (BL-042) — needs Theme 02 F002 + F003
7. **Theme 03, Feature 002**: clip-effect-model (BL-043 partial) — needs Theme 03 F001
8. **Theme 03, Feature 003**: text-overlay-apply (BL-043 partial) — needs Theme 03 F002

**No changes from Task 005.** Execution order validated — all dependencies confirmed by investigation.

---

## Backlog Coverage Verification

| Backlog | Feature(s) | Status |
|---------|-----------|--------|
| BL-037 | Theme 01, Feature 001 (expression-engine) | Covered |
| BL-038 | Theme 01, Feature 002 (graph-validation) | Covered |
| BL-039 | Theme 02, Feature 001 (filter-composition) | Covered |
| BL-040 | Theme 02, Feature 002 (drawtext-builder) | Covered |
| BL-041 | Theme 02, Feature 003 (speed-control) | Covered |
| BL-042 | Theme 03, Feature 001 (effect-discovery) | Covered |
| BL-043 | Theme 03, Features 002 + 003 (clip-effect-model + text-overlay-apply) | Covered |

All 7 backlog items covered. No deferrals. No descoping.

---

## Risk Resolutions Integrated

| Risk | Resolution | Design Impact |
|------|-----------|---------------|
| Clip effects storage | `effects_json TEXT` via audit log pattern; Rust Clip unchanged | Feature 002 scoping clarified |
| Effect registry pattern | Dictionary-based, following job handler registry pattern | Feature 001 design specified |
| Expression subset scope | Bounded whitelist: `{between, if, min, max}` + arithmetic + `{t, PTS, n}` | Feature 001 scoping clarified |
| Contract tests in CI | FFmpeg on all 9 CI entries; existing record-replay infra | No change needed |
| Drop-frame vs speed | PTS is frame-rate independent; drop-frame irrelevant | No change needed |

See `risk-assessment.md` for full details on all 8 risks.
