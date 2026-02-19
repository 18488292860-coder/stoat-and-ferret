# Version Design: v007

## Description

**Effect Workshop GUI** — Complete remaining effect types (audio mixing, transitions), build the effect registry with JSON schema validation and registry-based dispatch, and construct the full GUI effect workshop with catalog UI, parameter forms, and live preview. Covers milestones M2.4-2.6, M2.8-2.9.

9 backlog items (BL-044 through BL-052), organized into 4 themes with 11 features total (9 backlog + 2 documentation).

## Design Artifacts

Full design analysis available at: `comms/outbox/versions/design/v007/`

## Constraints and Assumptions

- v006 effects engine complete (filter expression engine, graph validation, text overlay, speed control, EffectRegistry)
- v005 GUI shell complete (React/TypeScript/Vite, Zustand stores, WebSocket, component patterns)
- Non-destructive editing: never modify source files
- Rust for compute, Python for orchestration
- Transparency: API responses include generated FFmpeg filter strings
- SPA fallback routing still deferred (LRN-023)

See `comms/outbox/versions/design/v007/001-environment/version-context.md` for full context.

## Key Design Decisions

- **DuckingPattern**: Builds a FilterGraph using composition API (compose_branch + compose_chain + compose_merge), not a single FilterChain
- **Two-input filters** (xfade, acrossfade): Use existing `FilterChain::new().input(label1).input(label2)` pattern
- **Registry refactoring**: Add `build_fn` to EffectDefinition, replace if/elif dispatch with `definition.build_fn(parameters)`
- **Effect CRUD**: Array-index-based (0-based) for v007 scope; UUID-based IDs as documented upgrade path
- **Preview thumbnails**: Deferred; filter string preview panel satisfies transparency intent
- **Custom form generator**: Lightweight for 5-6 parameter types; RJSF documented as upgrade path

See `comms/outbox/versions/design/v007/006-critical-thinking/risk-assessment.md` for rationale.

## Theme Overview

| # | Theme | Goal | Features | Backlog |
|---|-------|------|----------|---------|
| 1 | rust-filter-builders | Audio mixing and video transition Rust builders | 2 | BL-044, BL-045 |
| 2 | effect-registry-api | Registry refactor, transition API, architecture docs | 3 | BL-046, BL-047 |
| 3 | effect-workshop-gui | Catalog, forms, preview, workflow GUI | 4 | BL-048, BL-049, BL-050, BL-051 |
| 4 | quality-validation | E2E tests and API spec update | 2 | BL-052 |

See `THEME_INDEX.md` for details.

## Execution Order

```
T01 (Rust Builders) --> T02 (Registry & API) --> T03 (GUI Workshop) --> T04 (Quality)
```

Strictly sequential. Each theme depends on the previous theme's output.
