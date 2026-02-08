# v006 Gap Analysis: Effects Engine Foundation

**Milestones:** M2.1 (Filter Expression Engine), M2.2 (Text Overlay System), M2.3 (Speed Control)
**Design docs:** 01-roadmap.md, 05-api-specification.md (effects endpoints)

## Existing Coverage

No backlog items target v006. The existing FFmpeg filter system (v001) provides a foundation: `Filter`, `FilterChain`, `FilterGraph` types with helpers for `scale()`, `pad()`, `format()`, `concat()`. But the expression engine, text overlay, and speed control are entirely absent.

## Gaps Identified

### M2.1: Filter Expression Engine

**Gap 1: No expression parser/builder.** FFmpeg filter expressions (`enable='between(t,3,5)'`, alpha expressions, time-based expressions) have no Rust representation. Current filter system handles simple key=value parameters only.

**Gap 2: No filter graph validation.** Current `FilterGraph` builds strings but doesn't validate input/output pad matching. M2.1 requires compile-time-safe graph construction.

**Gap 3: No filter composition system.** No support for chaining, branching, or merging filter chains with verified pad connectivity.

**Gap 4: No property-based tests for filter expressions.** M2.1 specifies proptest for expression generation. Existing proptest coverage is limited to timeline math.

### M2.2: Text Overlay System

**Gap 5: No drawtext filter builder.** `escape_filter_text()` exists in Rust sanitize module, but no structured drawtext builder with position, color, shadow, box, animation parameters.

**Gap 6: No effect discovery API.** M2.2 specifies Python API for effect discovery with AI hints. No `/effects` endpoint exists.

**Gap 7: No alpha animation expressions.** Fade in/out for text overlays requires time-based alpha expressions, which depend on Gap 1.

### M2.3: Speed Control

**Gap 8: No setpts/atempo builders.** No Rust types for video speed (setpts) or audio speed (atempo chain for >2x).

**Gap 9: No audio removal option.** Speed control needs option to drop audio at extreme speeds.

## Suggested Backlog Items

### 1. Implement FFmpeg Filter Expression Engine in Rust
- **Priority:** P1 | **Tags:** `v006`, `rust`, `filters`, `expressions` | **Size:** L
- **Description:** Build expression parser/builder in Rust per M2.1. Support enable expressions (`between(t,start,end)`), alpha expressions, arithmetic expressions. Provide type-safe builder API exposed to Python via PyO3.
- **Acceptance criteria:**
  1. Expression types: enable, alpha, time, arithmetic
  2. Builder API prevents syntactically invalid expressions
  3. Expressions serialize to valid FFmpeg syntax
  4. Property-based tests (proptest) for expression generation
  5. PyO3 bindings with Python type stubs
- **Depends on:** None

### 2. Implement Filter Graph Validation
- **Priority:** P1 | **Tags:** `v006`, `rust`, `filters`, `validation` | **Size:** M
- **Description:** Add input/output pad matching validation to FilterGraph per M2.1. Verify chain connectivity before serialization.
- **Acceptance criteria:**
  1. Pad labels validated for matching (output feeds input)
  2. Unconnected pads detected and reported
  3. Graph cycles detected and rejected
  4. Validation errors include actionable messages
  5. Existing FilterGraph tests updated
- **Depends on:** None (enhances existing FilterGraph)

### 3. Build Filter Composition System
- **Priority:** P1 | **Tags:** `v006`, `rust`, `filters`, `composition` | **Size:** M
- **Description:** Create composition API for chaining, branching, and merging filter chains per M2.1. Enable building complex filter graphs programmatically.
- **Acceptance criteria:**
  1. Chain composition: sequential filter application
  2. Branch: split one stream into multiple
  3. Merge: combine multiple streams (e.g., overlay, amix)
  4. Composed graphs pass validation (Item 2)
  5. PyO3 bindings with Python type stubs
- **Depends on:** Item 2

### 4. Implement Drawtext Filter Builder
- **Priority:** P1 | **Tags:** `v006`, `rust`, `text-overlay` | **Size:** M
- **Description:** Build type-safe drawtext filter builder in Rust per M2.2. Support text content (using existing `escape_filter_text()`), position, font, color, shadow, box background, and alpha animation.
- **Acceptance criteria:**
  1. Position options: absolute, centered, margins
  2. Styling: font size, color, shadow offset/color, box background
  3. Alpha animation: fade in/out with configurable duration
  4. Generated filters validated as correct FFmpeg syntax
  5. Contract tests: generated commands pass `ffmpeg -filter_complex` validation
- **Depends on:** Item 1 (for alpha animation expressions)

### 5. Implement Speed Control Filters
- **Priority:** P1 | **Tags:** `v006`, `rust`, `speed-control` | **Size:** M
- **Description:** Build setpts (video) and atempo (audio) filter builders in Rust per M2.3. Handle audio chain generation for speeds >2x (atempo maxes at 2.0, requires chaining). Include audio removal option for extreme speeds.
- **Acceptance criteria:**
  1. Video speed via setpts with configurable factor (0.25x–4.0x)
  2. Audio speed via atempo with automatic chaining for >2x
  3. Option to drop audio instead of adjusting
  4. Validation with helpful error messages for out-of-range values
  5. Unit tests with edge cases (1x, boundary values, extreme speeds)
- **Depends on:** None

### 6. Create Effect Discovery API Endpoint
- **Priority:** P2 | **Tags:** `v006`, `api`, `effects`, `discovery` | **Size:** M
- **Description:** Implement `/effects` discovery endpoint per M2.2 and 05-api-specification.md. Return available effects with parameter schemas and AI hints. Lays foundation for v007 Effect Registry (M2.6).
- **Acceptance criteria:**
  1. GET `/effects` returns list of available effects
  2. Each effect includes: name, description, parameter JSON schema
  3. AI hints included for parameter guidance
  4. Text overlay and speed control registered as effects
  5. Response includes Rust-generated filter preview for default params
- **Depends on:** Items 4, 5

### 7. Apply Text Overlay to Clip API
- **Priority:** P2 | **Tags:** `v006`, `api`, `text-overlay` | **Size:** M
- **Description:** Create API endpoint to apply text overlay effect to a clip. Bridges Rust drawtext builder with project/clip model. Stores effect configuration on clip.
- **Acceptance criteria:**
  1. POST endpoint applies text overlay to clip
  2. Effect stored in clip/project model
  3. FFmpeg filter string generated and returned in response
  4. Validation errors surface Rust error messages
  5. Black box test covers apply → verify filter string flow
- **Depends on:** Items 4, 6
- **Note:** Requires extending clip model to store effects — may need EXP first.
