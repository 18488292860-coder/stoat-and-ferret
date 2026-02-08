# Backlog Items Created

## v004: Testing Infrastructure & Quality Verification (M1.8–1.9)

| ID | Title | Priority | Tags |
|----|-------|----------|------|
| BL-020 | Implement InMemory test doubles for Projects and Jobs | P1 | v004, testing, test-doubles |
| BL-021 | Add dependency injection to create_app() for test wiring | P1 | v004, testing, dependency-injection |
| BL-022 | Build fixture factory with builder pattern for test data | P1 | v004, testing, fixtures |
| BL-023 | Implement black box test scenario catalog | P1 | v004, testing, black-box |
| BL-024 | Contract tests with real FFmpeg for executor fidelity | P2 | v004, testing, ffmpeg, contract |
| BL-025 | Security audit of Rust path validation and input sanitization | P2 | v004, security, audit |
| BL-026 | Rust vs Python performance benchmark for core operations | P3 | v004, benchmarking, rust |
| BL-027 | Async job queue for scan operations | P2 | v004, async, scan |

**Existing items tagged v004:**
- BL-009: Add property test guidance to feature design template (P2)
- BL-010: Configure Rust code coverage with llvm-cov (P3)
- BL-012: Fix coverage reporting gaps for ImportError fallback (P3)
- BL-014: Add Docker-based local testing option (P2)
- BL-016: Unify InMemory vs FTS5 search behavior (P3)

## v005: GUI Shell, Library Browser & Project Manager (M1.10–1.12)

| ID | Title | Priority | Tags |
|----|-------|----------|------|
| BL-028 | EXP: Frontend framework selection and Vite project setup | P1 | v005, gui, investigation |
| BL-029 | Implement WebSocket endpoint for real-time events | P1 | v005, websocket, api |
| BL-030 | Build application shell and navigation components | P1 | v005, gui, shell |
| BL-031 | Build dashboard panel with health cards and activity log | P2 | v005, gui, dashboard |
| BL-032 | Implement thumbnail generation pipeline for video library | P1 | v005, thumbnails, ffmpeg |
| BL-033 | Build library browser with video grid, search, and scan UI | P1 | v005, gui, library |
| BL-034 | Fix pagination total count for list endpoints | P2 | v005, api, pagination |
| BL-035 | Build project manager with list, creation, and details views | P1 | v005, gui, projects |
| BL-036 | Set up Playwright E2E test infrastructure for GUI | P2 | v005, testing, e2e |

**Existing items in v005:**
- BL-003: EXP-003: FastAPI static file serving for GUI (P2)

## v006: Effects Engine Foundation (M2.1–2.3)

| ID | Title | Priority | Tags |
|----|-------|----------|------|
| BL-037 | Implement FFmpeg filter expression engine in Rust | P1 | v006, rust, filters, expressions |
| BL-038 | Implement filter graph validation for pad matching | P1 | v006, rust, filters, validation |
| BL-039 | Build filter composition system for chaining, branching, and merging | P1 | v006, rust, filters, composition |
| BL-040 | Implement drawtext filter builder for text overlays | P1 | v006, rust, text-overlay |
| BL-041 | Implement speed control filter builders (setpts/atempo) | P1 | v006, rust, speed-control |
| BL-042 | Create effect discovery API endpoint | P2 | v006, api, effects, discovery |
| BL-043 | Create API endpoint to apply text overlay effect to clips | P2 | v006, api, text-overlay |

## v007: Effect Workshop GUI (M2.4–2.6, 2.8–2.9)

| ID | Title | Priority | Tags |
|----|-------|----------|------|
| BL-044 | Implement audio mixing filter builders (amix/volume/fade) | P1 | v007, rust, audio, mixing |
| BL-045 | Implement transition filter builders (fade/xfade) | P1 | v007, rust, transitions |
| BL-046 | Create transition API endpoint for clip-to-clip transitions | P1 | v007, api, transitions |
| BL-047 | Build effect registry with JSON schema validation and builder protocol | P1 | v007, registry, effects, schema |
| BL-048 | Build effect catalog UI component | P1 | v007, gui, effects, catalog |
| BL-049 | Build dynamic parameter form generator from JSON schema | P1 | v007, gui, effects, forms |
| BL-050 | Implement live FFmpeg filter preview in effect parameter UI | P1 | v007, gui, preview, transparency |
| BL-051 | Build effect builder workflow with clip selector and effect stack | P1 | v007, gui, effects, builder |
| BL-052 | E2E tests for effect workshop workflow | P2 | v007, testing, e2e, effects |
