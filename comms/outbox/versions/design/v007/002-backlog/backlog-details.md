# Backlog Details - v007

## BL-044: Implement audio mixing filter builders (amix/volume/fade)

- **Priority:** P1 | **Size:** L | **Status:** open
- **Tags:** v007, rust, audio, mixing

**Description:**
> No Rust types exist for audio mixing, volume control, or audio fade effects. M2.4 requires amix for combining audio tracks, per-track volume control, fade in/out, and audio ducking patterns. The existing filter system handles video filters only. Without audio builders, mixing multiple audio sources (music + speech) requires manual FFmpeg filter string construction with no validation or ducking automation.

**Acceptance Criteria:**
1. amix filter builder supports configurable number of input tracks
2. Per-track volume control validates range 0.0-10.0 using existing validate_volume
3. Audio fade in/out supports configurable duration via expression engine
4. Audio ducking pattern lowers music volume during speech segments
5. Edge case tests cover silence, clipping prevention, and format mismatches

**Complexity:** High. Introduces an entirely new audio domain to the Rust filter system. Requires amix (multi-input), volume, afade builders plus a composite ducking pattern. Expression engine integration for fade curves adds complexity. Must handle audio-specific edge cases (silence, clipping, format mismatches) that don't exist in video filters.

---

## BL-045: Implement transition filter builders (fade/xfade)

- **Priority:** P1 | **Size:** M | **Status:** open
- **Tags:** v007, rust, transitions

**Description:**
> No Rust types exist for video transitions. M2.5 requires fade in, fade out, crossfade between clips, and xfade with selectable transition effects (wipeleft, slideright, etc.). Transitions are fundamental to video editing but currently have no type-safe builder. Without these, transition effects must be manually assembled as raw FFmpeg filter strings with no parameter validation.

**Acceptance Criteria:**
1. Fade in/out with configurable duration and color
2. Crossfade between two clips with overlap duration parameter
3. xfade supports selectable effect types (fade, wipeleft, slideright, etc.)
4. Parameter validation rejects invalid duration or effect type with helpful messages
5. PyO3 bindings expose transition builders to Python with type stubs

**Complexity:** Medium. Follows the proven builder pattern from v006. xfade's selectable transition types require an enum-based approach similar to position presets in DrawtextBuilder. Two-input filters (crossfade, xfade) are a new pattern compared to single-input filters in v006.

---

## BL-046: Create transition API endpoint for clip-to-clip transitions

- **Priority:** P1 | **Size:** M | **Status:** open
- **Tags:** v007, api, transitions

**Description:**
> M2.5 specifies an `/effects/transition` endpoint for applying transitions between adjacent clips, but no such endpoint exists. The Rust transition builders will generate filter strings, but there is no REST endpoint to receive transition parameters, validate clip adjacency, and store the transition in the project timeline. Without this, transitions cannot be applied through the API.

**Acceptance Criteria:**
1. POST endpoint applies a transition between two specified clips
2. Validates that the two clips are adjacent in the project timeline
3. Response includes the generated FFmpeg filter string
4. Transition configuration stored persistently in the project model
5. Black box test covers apply transition and verify filter output flow

**Complexity:** Medium. Similar pattern to the clip effect application endpoint from v006 T03. Main new challenge is clip adjacency validation in the timeline model and persistent storage of inter-clip (not intra-clip) effects.

---

## BL-047: Build effect registry with JSON schema validation and builder protocol

- **Priority:** P1 | **Size:** L | **Status:** open
- **Tags:** v007, registry, effects, schema

**Description:**
> M2.6 specifies a central effect registry where each effect is registered with its JSON schema, Rust validation functions, and builder protocol. The v006 discovery endpoint provides basic listing, but the full registry pattern — schema validation, effect builder injection, and metrics tracking — is missing. Without a registry, each effect is independently wired, making the Effect Workshop GUI unable to dynamically generate forms or validate parameters.

**Acceptance Criteria:**
1. Registry stores effect metadata, parameter JSON schemas, and builder references
2. JSON schema validation enforces parameter constraints for all registered effects
3. Effect builder protocol supports dependency injection for effect construction
4. All existing effects (text overlay, speed, audio, transitions) registered
5. effect_applications_total Prometheus counter increments by effect type on application

**Complexity:** High. This is the architectural centerpiece of v007. Must design a schema format that drives both backend validation and frontend form generation. Replaces the v006 if/elif dispatch with registry-based dispatch. The builder protocol and DI pattern are new abstractions. Prometheus metrics integration is straightforward but adds scope.

---

## BL-048: Build effect catalog UI component

- **Priority:** P1 | **Size:** M | **Status:** open
- **Tags:** v007, gui, effects, catalog

**Description:**
> M2.8 specifies an effect catalog component for browsing and selecting available effects, but no such frontend component exists. The `/effects` discovery endpoint (v006) provides the data, but there is no UI to consume it. Users need a visual catalog to discover what effects are available before they can configure and apply them. This is the entry point for the entire Effect Workshop workflow.

**Acceptance Criteria:**
1. Grid/list view displays available effects fetched from /effects endpoint
2. Effect cards show name, description, and category
3. AI hints displayed as contextual tooltips on each effect
4. Search and filter by category narrows the displayed effects
5. Clicking an effect opens its parameter configuration form

**Complexity:** Medium. Standard React component work consuming an existing API. Search/filter is client-side over a small dataset. The main design challenge is the visual layout (grid vs list toggle) and integration point with the parameter form (BL-049).

---

## BL-049: Build dynamic parameter form generator from JSON schema

- **Priority:** P1 | **Size:** L | **Status:** open
- **Tags:** v007, gui, effects, forms

**Description:**
> M2.8 specifies auto-generating parameter forms from JSON schema, but no dynamic form rendering component exists. Each effect has a unique parameter schema (from the effect registry), and building static forms per effect doesn't scale. Without a schema-driven form generator, every new effect requires custom frontend code for its parameter UI, and the live filter preview feature cannot work generically.

**Acceptance Criteria:**
1. Forms generated dynamically from effect JSON schema definitions
2. Supports parameter types: number (with range slider), string, enum (dropdown), boolean, color picker
3. Inline validation displays error messages from Rust validation
4. Live filter string preview updates as parameter values change
5. Default values pre-populated from the JSON schema

**Complexity:** High. Schema-to-form rendering is architecturally significant. Must handle diverse input types (sliders, dropdowns, color pickers), validation feedback from the backend, and real-time preview integration. The JSON schema format from BL-047 directly drives this component.

---

## BL-050: Implement live FFmpeg filter preview in effect parameter UI

- **Priority:** P1 | **Size:** M | **Status:** open
- **Tags:** v007, gui, preview, transparency

**Description:**
> M2.8 specifies showing the Rust-generated FFmpeg filter string in real time as users adjust effect parameters. This is the core transparency feature — users see exactly what commands Rust generates. No such preview component exists. Without live preview, the tool's key differentiator (transparency into FFmpeg command generation) is invisible during effect configuration.

**Acceptance Criteria:**
1. Filter string panel displays the current FFmpeg filter and updates on parameter change
2. API calls to get filter preview are debounced to avoid excessive requests
3. FFmpeg filter syntax displayed with syntax highlighting
4. Copy-to-clipboard button copies the filter string for external use

**Complexity:** Medium. The preview API call pattern (debounced fetch on parameter change) is well-understood. Syntax highlighting for FFmpeg filter strings is the main UI challenge. Tight coupling with BL-049's form state for real-time updates.

---

## BL-051: Build effect builder workflow with clip selector and effect stack

- **Priority:** P1 | **Size:** L | **Status:** open
- **Tags:** v007, gui, effects, builder

**Description:**
> M2.9 specifies a complete effect builder workflow: select effect, configure parameters, choose target clip, view the effect stack per clip, and edit/remove applied effects. No such workflow exists. Individual components (catalog, form, preview) need to be composed into a coherent workflow. Without this, users cannot complete the full loop of discovering, configuring, applying, and managing effects on clips.

**Acceptance Criteria:**
1. Apply to Clip workflow presents a clip selector from the current project
2. Effect stack visualization shows all effects applied to a selected clip in order
3. Preview thumbnail displays a static frame with the effect applied
4. Edit action re-opens parameter form with existing values for an applied effect
5. Remove action deletes an effect from a clip's effect stack with confirmation

**Complexity:** High. Composes BL-048, BL-049, and BL-050 into a multi-step workflow. Requires effect CRUD operations (edit/remove) that don't yet exist in the API. Preview thumbnails need frame extraction and effect application — a pipeline investigation item. State management across the workflow steps adds complexity.

---

## BL-052: E2E tests for effect workshop workflow

- **Priority:** P2 | **Size:** M | **Status:** open
- **Tags:** v007, testing, e2e, effects

**Description:**
> The effect workshop comprises multiple GUI components (catalog, form generator, preview, builder workflow) that must work together end-to-end. No E2E tests cover this workflow. The v005 Playwright infrastructure provides the foundation, but effect-specific test scenarios are needed. Without E2E coverage, regressions in the multi-step effect application workflow go undetected.

**Acceptance Criteria:**
1. E2E test browses effect catalog and selects an effect
2. E2E test configures parameters and verifies filter preview updates in real time
3. E2E test applies effect to a clip and verifies effect stack display
4. E2E test edits and removes an applied effect successfully
5. Accessibility checks (WCAG AA) pass for all form components

**Complexity:** Medium. Builds on v005 Playwright infrastructure. Tests are integration-heavy but follow established patterns. Accessibility testing adds scope but tools (axe-core) are already available. Depends on all other GUI items being complete.
