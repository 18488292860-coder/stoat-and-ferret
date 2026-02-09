# Version Context - v005

## Version Goals

**v005 - GUI Shell, Library Browser & Project Manager**

Build the frontend from scratch: create a frontend project with chosen framework, add WebSocket support for real-time events, build the application shell with navigation, implement library browser with thumbnail generation, and create the project manager. This completes Phase 1 (Milestones M1.10-1.12).

## Backlog Items

| ID | Priority | Title |
|----|----------|-------|
| BL-003 | - | EXP-003: FastAPI static file serving (existing prerequisite) |
| BL-028 | P1 | EXP: Frontend framework selection and Vite setup |
| BL-029 | P1 | Implement WebSocket endpoint for real-time events |
| BL-030 | P1 | Build application shell and navigation |
| BL-031 | P2 | Build dashboard panel |
| BL-032 | P1 | Implement thumbnail generation pipeline |
| BL-033 | P1 | Build library browser |
| BL-034 | P2 | Fix pagination total count |
| BL-035 | P1 | Build project manager |
| BL-036 | P2 | E2E test infrastructure |

**Total:** 10 items (7 P1, 3 P2)

## Dependencies

- **v004 (complete):** Black box tests validate GUI backend endpoints
- **BL-030 depends on BL-028:** Shell needs framework selected first
- **BL-033 depends on BL-028 + BL-032:** Library browser needs framework + thumbnails
- **BL-036 depends on BL-030 + BL-033 + BL-035:** E2E tests need all major UI components

## Pending Investigations

| ID | Question | Status |
|----|----------|--------|
| EXP-003 / BL-003 | FastAPI static file serving - GUI deployment from API server | pending |
| BL-028 | Frontend framework selection (extends EXP-003) | pending |

These investigations must complete before or as part of early v005 themes.

## Constraints and Assumptions

- Phase 1 completion version - all foundation work (v001-v004) is done
- Frontend is greenfield - no existing frontend code
- WebSocket support was deferred from v003
- Framework selection (BL-028) is an investigation item that will determine technology choices for the rest of v005
- Static file serving (EXP-003/BL-003) is a prerequisite for serving the frontend from the FastAPI backend

## Deferred Items to Be Aware Of

| Item | From | Rationale |
|------|------|-----------|
| Drop-frame timecode support | M1.2 | Complex; start with non-drop-frame only (TBD) |
| Rust coverage threshold at 90% | v004 | Currently at 75%, target 90% |
| Scan endpoint blackbox tests | v004 | Requires real FFmpeg |

## Version-Agnostic Items (Opportunistic)

| ID | Priority | Title |
|----|----------|-------|
| BL-011 | P3 | Consolidate Python/Rust build backends |
| BL-018 | P2 | Create C4 architecture documentation |
| BL-019 | P1 | Add Windows bash /dev/null guidance to AGENTS.md |
