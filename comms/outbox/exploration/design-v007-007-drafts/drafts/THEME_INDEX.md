# Theme Index: v007

## 01-rust-filter-builders

Implement Rust filter builders for audio mixing and video transitions, extending the proven v006 builder pattern to two new effect domains.

**Features:**

- 001-audio-mixing-builders: Build AmixBuilder, VolumeBuilder, AfadeBuilder, and DuckingPattern in Rust with PyO3 bindings
- 002-transition-filter-builders: Build FadeBuilder, XfadeBuilder with TransitionType enum and AcrossfadeBuilder in Rust with PyO3 bindings

## 02-effect-registry-api

Refactor the effect registry to use builder-protocol dispatch, add JSON schema validation, create the transition API endpoint, and document architectural changes.

**Features:**

- 001-effect-registry-refactor: Add build_fn to EffectDefinition, replace dispatch with registry lookup, add JSON schema validation and Prometheus metrics
- 002-transition-api-endpoint: Create POST /effects/transition endpoint with clip adjacency validation and persistent storage
- 003-architecture-documentation: Update architecture and API specification docs to reflect registry refactoring and new endpoints

## 03-effect-workshop-gui

Build the complete Effect Workshop GUI: catalog, schema-driven forms, live preview, and full builder workflow.

**Features:**

- 001-effect-catalog-ui: Build grid/list view of available effects with search, filter, and AI hint tooltips
- 002-dynamic-parameter-forms: Build schema-driven form generator supporting number/range, string, enum, boolean, and color picker inputs
- 003-live-filter-preview: Build debounced filter string preview panel with syntax highlighting and copy-to-clipboard
- 004-effect-builder-workflow: Compose catalog, form, and preview into full workflow with clip selector, effect stack, and CRUD

## 04-quality-validation

Validate the complete effect workshop through E2E testing with accessibility compliance and update API specification documentation.

**Features:**

- 001-e2e-effect-workshop-tests: E2E tests covering catalog browse, parameter config, preview, apply/edit/remove, and WCAG AA accessibility
- 002-api-specification-update: Update OpenAPI spec and design docs with transition, preview, and CRUD endpoints
