# v007 Gap Analysis: Effect Workshop GUI

**Milestones:** M2.4 (Audio Mixing), M2.5 (Transitions), M2.6 (Effect Registry & Discovery), M2.8 (Effect Discovery UI), M2.9 (Effect Builder)
**Design docs:** 01-roadmap.md, 08-gui-architecture.md (Phase 2 components)

## Existing Coverage

No backlog items target v007. The entire effects GUI layer and remaining effects (audio, transitions) are unrepresented.

## Gaps Identified

### M2.4: Audio Mixing

**Gap 1: No amix filter builder.** No Rust types for audio mixing, volume control, fade, or ducking.

### M2.5: Transitions

**Gap 2: No fade/xfade filter builders.** No Rust types for video transitions (fade, crossfade, xfade effects).

**Gap 3: No transition API endpoint.** M2.5 specifies `/effects/transition` endpoint.

### M2.6: Effect Registry & Discovery

**Gap 4: No effect registry.** M2.6 specifies a registry with JSON schema validation, parameter validation as Rust functions, and effect builder protocol. The v006 discovery endpoint (suggested) would be a foundation, but the full registry pattern is missing.

**Gap 5: No effect metrics.** M2.6 specifies `effect_applications_total` Prometheus counter by effect type.

### M2.8–2.9: GUI Effect Workshop

**Gap 6: No effect catalog UI.** No frontend component to browse and select effects.

**Gap 7: No parameter form generator.** M2.8 specifies auto-generating forms from JSON schema — requires both schema infrastructure and dynamic form rendering.

**Gap 8: No live filter preview.** M2.8 specifies showing the Rust-generated FFmpeg filter string in real time as users adjust parameters. Core differentiator for transparency.

**Gap 9: No effect builder workflow.** M2.9 specifies clip selector, effect stack visualization, preview thumbnail, and edit/remove actions.

## Suggested Backlog Items

### 1. Implement Audio Mixing Filter Builders
- **Priority:** P1 | **Tags:** `v007`, `rust`, `audio`, `mixing` | **Size:** M
- **Description:** Build amix, volume, and audio fade filter builders in Rust per M2.4. Support per-track volume control, fade in/out, and audio ducking patterns.
- **Acceptance criteria:**
  1. amix filter with configurable number of inputs
  2. Per-track volume control (0.0–10.0, using existing validate_volume)
  3. Audio fade in/out with configurable duration
  4. Audio ducking pattern (lower music during speech)
  5. Edge case tests: silence, clipping, format mismatches
- **Depends on:** v006 Item 1 (expression engine for fade expressions)

### 2. Implement Transition Filter Builders
- **Priority:** P1 | **Tags:** `v007`, `rust`, `transitions` | **Size:** M
- **Description:** Build fade and xfade filter builders in Rust per M2.5. Support fade in, fade out, crossfade between clips, and xfade transition effects.
- **Acceptance criteria:**
  1. Fade in/out with configurable duration and color
  2. Crossfade between two clips with overlap duration
  3. xfade with selectable effect type (fade, wipeleft, slideright, etc.)
  4. Parameter validation with helpful error messages
  5. PyO3 bindings with Python type stubs
- **Depends on:** v006 Item 1 (expression engine)

### 3. Create Transition API Endpoint
- **Priority:** P1 | **Tags:** `v007`, `api`, `transitions` | **Size:** M
- **Description:** Implement `/effects/transition` endpoint per M2.5. Apply transitions between adjacent clips in a project timeline.
- **Acceptance criteria:**
  1. POST endpoint applies transition between two clips
  2. Validates clips are adjacent in timeline
  3. Returns generated FFmpeg filter string
  4. Transition stored in project model
  5. Black box test covers apply and verify flow
- **Depends on:** Item 2, v006 Item 7 (effect-on-clip model)

### 4. Build Effect Registry with JSON Schema Validation
- **Priority:** P1 | **Tags:** `v007`, `registry`, `effects`, `schema` | **Size:** L
- **Description:** Design and implement effect registry per M2.6. Each effect registered with JSON schema for parameters, Rust validation functions, and builder protocol. Extends v006 discovery endpoint.
- **Acceptance criteria:**
  1. Registry stores effect metadata, parameter schemas, and builders
  2. JSON schema validation for all effect parameters
  3. Effect builder protocol (dependency injection pattern)
  4. All existing effects (text, speed, audio, transitions) registered
  5. `effect_applications_total` Prometheus counter by effect type
- **Depends on:** Items 1, 2, v006 Items 4–6
- **Note:** May need EXP to design registry schema and builder protocol.

### 5. Build Effect Catalog UI
- **Priority:** P1 | **Tags:** `v007`, `gui`, `effects`, `catalog` | **Size:** M
- **Description:** Create effect catalog component per M2.8. Browse available effects from `/effects` endpoint, display AI hints as tooltips, and show parameter descriptions.
- **Acceptance criteria:**
  1. Grid/list of available effects fetched from API
  2. Effect cards show name, description, category
  3. AI hints displayed as contextual tooltips
  4. Search/filter by category
  5. Click to open parameter form
- **Depends on:** Item 4, v005 Item 1 (frontend project)

### 6. Build Dynamic Parameter Form Generator
- **Priority:** P1 | **Tags:** `v007`, `gui`, `effects`, `forms` | **Size:** L
- **Description:** Create component that auto-generates parameter forms from JSON schema per M2.8. Support number, string, enum, boolean, and color parameter types with inline validation.
- **Acceptance criteria:**
  1. Forms generated from effect JSON schema
  2. Parameter types: number (with range), string, enum (dropdown), boolean, color picker
  3. Inline validation with error messages from Rust
  4. Live filter string preview updates as parameters change
  5. Default values pre-populated from schema
- **Depends on:** Item 5

### 7. Implement Live Filter Preview
- **Priority:** P1 | **Tags:** `v007`, `gui`, `preview`, `transparency` | **Size:** M
- **Description:** Show real-time FFmpeg filter string as users adjust effect parameters per M2.8. Core transparency feature — users see exactly what Rust generates.
- **Acceptance criteria:**
  1. Filter string displayed and updates on parameter change
  2. Debounced API calls to get filter preview
  3. Syntax-highlighted FFmpeg filter display
  4. Copy-to-clipboard functionality
- **Depends on:** Item 6

### 8. Build Effect Builder Workflow
- **Priority:** P1 | **Tags:** `v007`, `gui`, `effects`, `builder` | **Size:** L
- **Description:** Create effect builder per M2.9: configure effect, select target clip, view effect stack per clip, preview thumbnail with effect applied, and edit/remove actions.
- **Acceptance criteria:**
  1. "Apply to Clip" workflow with clip selector
  2. Effect stack visualization showing all effects on a clip
  3. Effect preview thumbnail (static frame with effect applied)
  4. Edit existing effect parameters
  5. Remove effect from clip
- **Depends on:** Items 5, 6, 7
- **Note:** Preview thumbnail requires backend frame extraction + effect application — may need EXP.

### 9. E2E Tests for Effect Workshop
- **Priority:** P2 | **Tags:** `v007`, `testing`, `e2e`, `effects` | **Size:** M
- **Description:** Playwright E2E tests for effect workshop workflow. Extends v005 E2E infrastructure to cover effect discovery, configuration, and application.
- **Acceptance criteria:**
  1. Browse effect catalog and select an effect
  2. Configure parameters and verify filter preview updates
  3. Apply effect to clip and verify effect stack
  4. Edit and remove an effect
  5. Accessibility checks for form components
- **Depends on:** Item 8, v005 Item 9 (E2E infrastructure)
