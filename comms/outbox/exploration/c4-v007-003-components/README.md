# Task 003: Component Synthesis -- v007 Delta Results

## Summary

Synthesized 42 code-level documentation files into 8 logical components for the stoat-and-ferret C4 architecture documentation. This is a delta update from the v006 baseline; the 8-component structure remains sound with content updates reflecting v007 feature additions.

- **Component count**: 8 (unchanged from v006)
- **Code-level files mapped**: 42 (was 41; added `c4-code-gui-e2e`)
- **Files modified**: 8 component docs + 42 code-level parent links + 1 master index = 51 files total
- **New code-level file**: `c4-code-gui-e2e.md` (Playwright E2E tests) assigned to Web GUI

## Component Inventory

| # | Component | Files | Key v007 Changes |
|---|-----------|-------|------------------|
| 1 | Rust Core Engine | 8 | Added audio builders (VolumeBuilder, AfadeBuilder, AmixBuilder, DuckingPattern) and transition builders (FadeBuilder, XfadeBuilder, AcrossfadeBuilder, 59 TransitionType variants) |
| 2 | Python Bindings Layer | 3 | Updated re-exports to include all audio/transition builder types |
| 3 | Effects Engine | 1 | Expanded from 2 to 9 built-in effects; added EffectValidationError, jsonschema validation |
| 4 | API Gateway | 7 | Added effects CRUD endpoints (apply/update/remove at index), transition application endpoint |
| 5 | Application Services | 3 | Added Rust Command Bridge to features; minor description updates |
| 6 | Data Access Layer | 3 | Clarified effects/transitions stored as JSON in Clip/Project models; removed incorrect SQLAlchemy/ORM references |
| 7 | Web GUI | 8 | Major expansion: 4 routes (added Effects), 22 components (was 18), 8 hooks (was 6), 7 stores (was 3), 15 Playwright E2E tests (new) |
| 8 | Test Infrastructure | 9 | Updated test counts (~857 total); added audio/transition builder parity tests |

## Boundary Decisions

### Unchanged Boundaries (8 components retained from v006)
The v007 feature additions (audio/transition builders, full effect CRUD, effect workshop GUI) fit naturally within the existing component boundaries without requiring structural changes.

### New File Assignment
- **`c4-code-gui-e2e.md`** (Playwright E2E tests) was assigned to **Web GUI** rather than Test Infrastructure because these are browser-level tests specific to the React SPA. The Python backend test suites under `tests/` remain in Test Infrastructure. This preserves the boundary principle that GUI test code co-locates with GUI source code.

### Boundary Rationale
1. **Rust Core Engine** (8 files): All Rust source code in `rust/stoat_ferret_core/src/`. Pure computation with no I/O. Clear language boundary.
2. **Python Bindings Layer** (3 files): Bridge code between Rust and Python. Package re-exports, type stubs, and verification scripts. Exists solely to make Rust types available to Python.
3. **Effects Engine** (1 file): Self-contained effect registry with build functions, schemas, and AI hints. Has its own module under `src/stoat_ferret/effects/`. Consumed by API but not part of HTTP routing.
4. **API Gateway** (7 files): All HTTP/WebSocket routing, middleware, request/response schemas, and application wiring. Boundary follows the `src/stoat_ferret/api/` directory minus the `services/` subdirectory.
5. **Application Services** (3 files): Business logic decoupled from HTTP. Scan service, FFmpeg execution, job queue. The `services/` subdirectory of `api/` plus standalone `ffmpeg/` and `jobs/` modules.
6. **Data Access Layer** (3 files): All persistence concerns. Repository protocols/implementations, domain models, schema, audit logging, and package root (version/logging). Boundary follows `src/stoat_ferret/db/` plus the package `__init__.py`.
7. **Web GUI** (8 files): Everything under `gui/`. React components, hooks, pages, stores, Vitest unit tests, and Playwright E2E tests. Clear technology boundary (TypeScript/React vs Python).
8. **Test Infrastructure** (9 files): All Python test suites under `tests/`. Organized by test type (unit, API, blackbox, contract, security, coverage, doubles, jobs, property-based).

## Cross-Component Dependencies

```
Web GUI --> API Gateway (HTTP/REST + WebSocket)
API Gateway --> Effects Engine (list/apply/update/remove effects)
API Gateway --> Application Services (delegates business logic)
API Gateway --> Data Access Layer (reads/writes via repositories)
API Gateway --> Python Bindings (clip validation, transitions)
Effects Engine --> Python Bindings (Rust builders for all 9 effects)
Application Services --> Data Access Layer (persists scanned data)
Application Services --> Python Bindings (FFmpegCommand bridge)
Data Access Layer --> Python Bindings (Clip.validate())
Python Bindings --> Rust Core Engine (imports compiled _core module)
Test Infrastructure --> API Gateway, Application Services, Data Access, Python Bindings (tests)
```

## Delta Changes vs v006

### Content Updates (no structural changes)
- **Rust Core Engine**: Added audio module (4 builder types) and transitions module (3 builder types + TransitionType enum with 59 variants) to description, features, interfaces, and Mermaid diagram
- **Python Bindings**: Updated description to mention audio/transition builder re-exports
- **Effects Engine**: Changed from "2 built-in effects (text_overlay, speed_control)" to "9 built-in effects" with full list; added EffectValidationError and jsonschema validation dependency; updated Mermaid diagram to show all 9 Rust builder integrations
- **API Gateway**: Added effects CRUD endpoints (POST/PATCH/DELETE for clip effects), transition application endpoint, and updated interface documentation
- **Data Access Layer**: Removed incorrect SQLAlchemy/ORM references (code uses raw sqlite3/aiosqlite); clarified effects and transitions stored as JSON fields
- **Web GUI**: Major content update -- 4 routes (added Effects), 22 components (added EffectCatalog, EffectParameterForm, FilterPreview, ClipSelector, EffectStack), 8 hooks (added useEffects, useEffectPreview), 7 stores (added effectCatalog, effectForm, effectPreview, effectStack), new E2E test file, added Playwright to technology stack, updated test counts (131 unit + 15 E2E)
- **Test Infrastructure**: Updated aggregate test counts; added audio builder (~42 tests) and transition builder (~46 tests) parity testing mentions

### New Mapping
- `c4-code-gui-e2e` --> Web GUI (new file in v007)

### Parent Component Links
- Updated all 42 code-level files with correct `Parent Component` links (previously 26 files were missing the field and 14 had `TBD`)

### Master Index
- Updated `c4-component.md` with: revised descriptions, updated Mermaid diagram (9 effects, audio/transitions, ~857 tests), new `c4-code-gui-e2e` row in mapping table, Web GUI file count changed from 7 to 8

## Files Modified

### Component Documentation (8 files)
- `docs/C4-Documentation/c4-component-rust-core-engine.md`
- `docs/C4-Documentation/c4-component-python-bindings.md`
- `docs/C4-Documentation/c4-component-effects-engine.md`
- `docs/C4-Documentation/c4-component-api-gateway.md`
- `docs/C4-Documentation/c4-component-application-services.md`
- `docs/C4-Documentation/c4-component-data-access.md`
- `docs/C4-Documentation/c4-component-web-gui.md`
- `docs/C4-Documentation/c4-component-test-infrastructure.md`

### Master Index (1 file)
- `docs/C4-Documentation/c4-component.md`

### Code-Level Parent Links (42 files)
All `docs/C4-Documentation/c4-code-*.md` files updated with `Parent Component` links.
