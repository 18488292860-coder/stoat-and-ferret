# C4 Component Synthesis — v006 Delta (Task 003)

The v006 delta introduced 6 new code-level documentation files covering changes across four existing concern areas plus one entirely new domain. The synthesis produced 8 components total — one net new component (Effects Engine) was added to accommodate the effects module, which represents a distinct domain boundary from the existing Application Services and API Gateway components. The main boundary decision requiring care was whether python-effects belonged to Application Services (execution focus) or stood alone; the effects module's sole responsibility is effect definition and discovery with no execution logic, justifying its own component.

## Components Identified: 8

1. **Rust Core Engine** — Updated to include c4-code-rust-core and c4-code-rust-ffmpeg (the expanded FFmpeg module with filter graph, expression trees, DrawtextBuilder, SpeedControl)
2. **Python Bindings Layer** — Unchanged; no delta code docs assigned here
3. **Effects Engine** — NEW component for c4-code-python-effects (EffectDefinition, EffectRegistry, create_default_registry)
4. **API Gateway** — Updated to include c4-code-python-api and c4-code-python-schemas alongside existing routers, middleware, websocket docs
5. **Application Services** — Unchanged; no delta code docs assigned here
6. **Data Access Layer** — Updated to include c4-code-python-db (SQLAlchemy ORM models and repository classes)
7. **Web GUI** — Unchanged; no delta code docs assigned here
8. **Test Infrastructure** — Unchanged; no delta code docs assigned here

## Code-to-Component Mapping

| New Delta Code File | Assigned Component | Rationale |
|--------------------|--------------------|-----------|
| c4-code-rust-core | Rust Core Engine | lib.rs is the crate root and module registration entry point |
| c4-code-rust-ffmpeg | Rust Core Engine | FFmpeg builders are part of the Rust compute kernel |
| c4-code-python-effects | Effects Engine (NEW) | Distinct domain: effect catalogue, not HTTP or execution |
| c4-code-python-api | API Gateway | Higher-level view of the same FastAPI application layer |
| c4-code-python-schemas | API Gateway | Pydantic schemas are the API Gateway's request/response contracts |
| c4-code-python-db | Data Access Layer | SQLAlchemy ORM models and repository pattern for data persistence |

## Boundary Rationale

**Effects Engine as its own component**: The effects module (`src/stoat_ferret/effects/`) has a single responsibility — defining the catalogue of available effects with their parameter schemas and AI hints. It does not execute effects (that is Application Services / FFmpeg Executor territory) and does not handle HTTP (that is API Gateway). Its only upward dependency is on the Rust Core Engine for preview function generation. A dedicated component accurately models this domain boundary and keeps Application Services focused on execution orchestration.

**c4-code-python-api and c4-code-python-schemas into API Gateway**: These new delta docs cover the same source directories as existing API Gateway code docs (c4-code-stoat-ferret-api, c4-code-stoat-ferret-api-routers, etc.) but at a different granularity level. Assigning them to the same component avoids a spurious split — they describe the HTTP presentation layer, which is the API Gateway's responsibility.

**c4-code-python-db into Data Access Layer**: The SQLAlchemy ORM models and repository classes document the same persistence domain as the existing c4-code-stoat-ferret-db. Different persistence technologies (aiosqlite direct vs SQLAlchemy ORM) within the same data access concern belong in the same component.

**c4-code-rust-core and c4-code-rust-ffmpeg into Rust Core Engine**: The rust-core doc covers lib.rs (module root and registration) and rust-ffmpeg covers the expanded ffmpeg module. Both are Rust compute kernel material, consistent with the existing Rust Core Engine composition.

## Cross-Component Dependencies

Key dependency flows introduced or clarified in v006:

- **Effects Engine → Rust Core Engine**: Preview functions call DrawtextBuilder and SpeedControl from the compiled _core extension. This is a downward dependency from Python domain logic to Rust compute.
- **API Gateway → Effects Engine**: The effects router calls EffectRegistry.list_all() and EffectRegistry.get() at request time. The registry is injected via the application factory.
- **Rust Core Engine → (none)**: Both new Rust docs confirm the engine remains a leaf component with no application-layer dependencies.

The overall dependency graph remains acyclic: GUI → API Gateway → Effects Engine/Application Services/Data Access Layer → Python Bindings → Rust Core Engine.

## Delta Changes vs Previous Component Structure

| Change | Detail |
|--------|--------|
| NEW component: Effects Engine | Introduced to model the entirely new effects module in v006 |
| Rust Core Engine: expanded | Added c4-code-rust-core and c4-code-rust-ffmpeg; component now covers 8 code docs instead of 6 |
| API Gateway: expanded | Added c4-code-python-api and c4-code-python-schemas; now 7 code docs, effects router documented |
| Data Access Layer: expanded | Added c4-code-python-db (SQLAlchemy ORM layer); now 3 code docs |
| Master index: updated | New row for Effects Engine, updated counts for changed components, full mapping table updated |
| Parent Component links: set | All 6 new delta code docs now have Parent Component links (previously TBD) |

## Files Written

### Updated in docs/C4-Documentation/
- `c4-component-rust-core-engine.md` — Added c4-code-rust-core and c4-code-rust-ffmpeg; expanded FFmpeg module features
- `c4-component-effects-engine.md` — NEW file
- `c4-component-api-gateway.md` — Added c4-code-python-api and c4-code-python-schemas; added effects endpoints
- `c4-component-data-access.md` — Added c4-code-python-db; added ORM models feature
- `c4-component.md` — Updated master index with 8 components and full mapping table

### Parent Component links set in:
- `c4-code-rust-core.md` — Rust Core Engine
- `c4-code-rust-ffmpeg.md` — Rust Core Engine
- `c4-code-python-effects.md` — Effects Engine
- `c4-code-python-api.md` — API Gateway
- `c4-code-python-schemas.md` — API Gateway
- `c4-code-python-db.md` — Data Access Layer
