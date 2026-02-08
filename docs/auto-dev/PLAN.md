# stoat-and-ferret - Development Plan

> Bridge between strategic roadmap and auto-dev execution.
> 
> Strategic Roadmap: `docs/design/01-roadmap.md`
> Last Updated: 2025-01-25

## Roadmap → Version Mapping

| Version | Roadmap Reference | Focus | Prerequisites | Status |
|---------|-------------------|-------|---------------|--------|
| v001 | Phase 1, M1.1-1.3 | Foundation + Rust core basics | EXP-001, EXP-002 | complete |
| v002 | Phase 1, M1.4-1.5 | Database + FFmpeg integration | v001 | complete |
| v003 | Phase 1, M1.6-1.7 | API layer + Clip model | v002 | complete |
| v004 | Phase 1, M1.8-1.9 | Testing infrastructure + quality verification | v003 | planned |
| v005 | Phase 1, M1.10-1.12 | GUI shell + library browser + project manager | v004 | planned |
| v006 | Phase 2, M2.1-2.3 | Effects engine foundation | v005 | planned |
| v007 | Phase 2, M2.4-2.6 | Effect Workshop GUI | v006 | planned |

## Investigation Dependencies

Track explorations that must complete before version design.

| ID | Question | Informs | Status | Results |
|----|----------|---------|--------|---------|
| EXP-001 | PyO3/maturin hybrid build workflow — dev experience, CI setup, stub generation | v001 | complete | [rust-python-hybrid](../../comms/outbox/exploration/rust-python-hybrid/) |
| EXP-002 | Recording fake pattern — concrete implementation for RecordingFFmpegExecutor | v001, v004 | complete | [recording-fake-pattern](../../comms/outbox/exploration/recording-fake-pattern/) |
| EXP-003 | FastAPI static file serving — GUI deployment from API server | v005 | pending | - |

## Scoping Decisions

### v001 Boundary

**Included:** Milestones 1.1, 1.2, 1.3
- Project setup with Python + Rust tooling
- Rust core: timeline math, validation, FFmpeg command builder
- PyO3 bindings with type stubs

**Rationale:** These form the foundation that everything else builds on. The Rust core must be working before any Python integration can proceed meaningfully.

**Deferred:** Database (M1.4) requires stable Rust types to define schemas.

### v002 Boundary

**Included:** Milestones 1.4, 1.5
- SQLite with repository pattern
- FFmpeg executor with dependency injection
- Integration between Rust command builder and Python executor

**Rationale:** Database and FFmpeg integration share similar testing patterns (recording fakes, contract tests). Natural to develop together.

### v003 Boundary

**Included:** Milestones 1.6, 1.7
- FastAPI REST endpoints
- Request correlation, metrics middleware
- Clip model using Rust timeline math

**Rationale:** API layer is the primary interface. Must be solid before building testing infrastructure around it.

### v004 Boundary

**Included:** Milestones 1.8, 1.9
- Black box testing harness
- Recording test doubles
- Contract tests for FFmpeg commands
- Quality verification suite

**Rationale:** Testing infrastructure is substantial enough to warrant its own version. Builds on stable API from v003.

### v005 Boundary

**Included:** Milestones 1.10, 1.11, 1.12
- GUI shell with React/Svelte + Vite
- Library browser component
- Project manager component

**Rationale:** GUI milestones are tightly coupled — shell provides frame for browser and manager.

### v006 Boundary

**Included:** Milestones 2.1, 2.2, 2.3
- FFmpeg filter expression engine (Rust)
- Filter graph validation and composition
- Text overlay system (drawtext builder)
- Speed control filters (setpts/atempo)
- Effect discovery API endpoint

**Rationale:** All effects engine work is pure Rust + API. Independent of v005 GUI — can be developed in parallel if needed. Lays the foundation for v007 Effect Workshop.

### v007 Boundary

**Included:** Milestones 2.4, 2.5, 2.6, 2.8, 2.9
- Audio mixing filter builders
- Transition filter builders and API
- Effect registry with JSON schema validation
- Effect catalog, parameter form generator, and live preview UI
- Effect builder workflow (apply, edit, remove effects on clips)

**Rationale:** Combines remaining effects (audio, transitions) with the full GUI Effect Workshop. M2.8–2.9 (GUI) depends on M2.4–2.6 (effects) being complete within the same version.

## Deferred Items

Items explicitly deferred during version design, with target versions.

| Item | From | To | Rationale |
|------|------|----|-----------|
| Structured logging with correlation ID | M1.1 | v003 | Requires API layer for request context |
| Prometheus metrics (/metrics endpoint) | M1.1 | v003 | Requires API layer |
| Health check endpoints (/health/*) | M1.1 | v003 | Requires API layer |
| Externalized settings (pydantic-settings) | M1.1 | v003 | Requires API layer for configuration |
| Project duration calculations | M1.2 | v003 | Requires Clip model |
| Encoding preset builder | M1.3 | v002 | Needed for FFmpeg executor integration |
| Contract tests with real FFmpeg | M1.3 | v004 | Part of testing infrastructure version |
| Drop-frame timecode support | M1.2 | TBD | Complex; start with non-drop-frame only |

## Completed Versions

| Version | Date | Summary |
|---------|------|---------|
| v001 | 2026-01-26 | Foundation version: Python/Rust tooling, timeline math, FFmpeg command builder |
| v002 | 2026-01-27 | Database & FFmpeg integration: Python bindings completion, SQLite with repository pattern, FFmpeg executor with observability |
| v003 | 2026-01-28 | API layer + Clip model: FastAPI REST API, async repository layer, video library endpoints, clip/project data models with Rust validation, CI improvements (BL-013, BL-015, BL-017) |

## Backlog Integration

Work that surfaces during planning or execution but doesn't fit current scope:

| Tag | Purpose |
|-----|---------|
| `investigation` | Needs exploration before implementation |
| `deferred` | Known work explicitly pushed to later |
| `discovered` | Found during execution, not originally planned |
| `blocked` | Waiting on external dependency |

### Backlog Coverage by Version (as of 2026-02-08)

Gap analysis completed for v004–v007. All planned versions now have backlog items covering their milestone scope.

| Version | Milestones | New Items | Existing Items Tagged | Total Coverage |
|---------|-----------|-----------|----------------------|----------------|
| v004 | M1.8–1.9 | 8 (BL-020–027) | 5 (BL-009, 010, 012, 014, 016) | 13 items |
| v005 | M1.10–1.12 | 9 (BL-028–036) | 1 (BL-003) | 10 items |
| v006 | M2.1–2.3 | 7 (BL-037–043) | 0 | 7 items |
| v007 | M2.4–2.6, 2.8–2.9 | 9 (BL-044–052) | 0 | 9 items |

**Cross-version dependencies:**
- v005 depends on v004 (black box testing infrastructure validates GUI backend)
- v006 is independent of v005 (pure Rust + API work)
- v007 depends on both v006 (effects engine) and v005 (frontend project)

**Items requiring exploration (EXP) before implementation:**
- BL-028: Frontend framework selection (extends BL-003)
- BL-043: Clip effect model design (how effects attach to clips)
- BL-047: Effect registry schema and builder protocol design
- BL-051: Preview thumbnail pipeline (frame extraction + effect application)

## Change Log

| Date | Change | Rationale |
|------|--------|-----------|
| 2025-01-25 | Initial plan created | Project bootstrap, mapping Phase 1-2 milestones to versions |
| 2025-01-25 | EXP-001, EXP-002 complete | Investigations for v001 prerequisites |
| 2025-01-25 | v001 design complete | 3 themes, 10 features designed |
| 2026-01-26 | v001 complete | Foundation version delivered |
| 2026-01-27 | v002 complete | Database & FFmpeg integration delivered (4 themes, 13 features) |
| 2026-01-28 | v003 complete | API layer + Clip model delivered (4 themes, 15 features, BL-013/015/017 completed) |
