# Theme Index: v006

## Theme 01: filter-engine

Build the foundational Rust filter infrastructure — expression type system, filter graph validation, and composition API. Corresponds to M2.1 (Filter Expression Engine).

**Backlog Items:** BL-037, BL-038, BL-039

**Dependencies:** None (first theme)

**Features:**

- 001-expression-engine: Type-safe Rust expression builder for FFmpeg filter expressions with proptest validation
- 002-graph-validation: Validate FilterGraph pad matching, detect unconnected pads and cycles using Kahn's algorithm
- 003-filter-composition: Programmatic chain, branch, and merge composition with automatic pad label management

---

## Theme 02: filter-builders

Implement concrete filter builders for text overlay and speed control using the expression engine from Theme 01. Corresponds to M2.2 (Text Overlay) and M2.3 (Speed Control).

**Backlog Items:** BL-040, BL-041

**Dependencies:** Theme 01 (expression engine)

**Features:**

- 001-drawtext-builder: Type-safe drawtext filter builder with position presets, styling, and alpha animation via expression engine
- 002-speed-builders: setpts and atempo filter builders with automatic atempo chaining for speeds above 2.0x

---

## Theme 03: effects-api

Create the Python-side effect registry, discovery API endpoint, clip effect application endpoint, and update architecture documentation. Bridges the Rust filter engine with the REST API and data model.

**Backlog Items:** BL-042, BL-043, Impact #2

**Dependencies:** Theme 02 (both filter builder types must be complete for registration)

**Features:**

- 001-effect-discovery: Effect registry with parameter schemas, AI hints, and GET /effects endpoint
- 002-clip-effect-api: POST endpoint to apply text overlay to clips with effect storage in clip model
- 003-architecture-docs: Update 02-architecture.md with new Rust modules, Effects Service, and clip model extension
