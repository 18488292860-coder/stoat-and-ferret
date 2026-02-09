# Retrospective Insights — v004 to v005

## What Worked Well (Continue)

### Infrastructure-First Sequencing
Theme 01's test foundation (InMemory doubles, DI, fixtures) was consumed by all 4 subsequent themes. **Apply to v005:** Scaffold the frontend project (BL-028) and WebSocket endpoint (BL-029) before building GUI panels that depend on them.

### Constructor DI via create_app()
The `create_app()` DI pattern eliminated global mutable state and made testing clean. **Apply to v005:** Extend this pattern for WebSocket manager injection and any new middleware.

### Handoff Documents Between Sequential Features
Theme 03's `handoff-to-next.md` provided clear context for each subsequent feature. **Apply to v005:** Use handoff docs between sequential features (e.g., frontend scaffolding → shell → dashboard).

### Record-Replay Testing for External Dependencies
The RecordingFFmpegExecutor/FakeFFmpegExecutor pattern proved reliable. **Apply to v005:** BL-032 (thumbnail pipeline) should use this same pattern for FFmpeg thumbnail extraction testing.

### Template-Driven Process Improvements
Embedding guidance in templates rather than standalone docs ensured adoption. **Apply to v005:** Any frontend conventions should be embedded in component templates, not separate docs.

## What Didn't Work (Avoid)

### Unachievable Acceptance Criteria
Theme 01 Feature 3 had `with_text_overlay()` as AC, but the domain model didn't exist. **Avoid in v005:** Validate all AC against the actual codebase during design. Especially relevant for GUI items referencing backend APIs — confirm endpoints exist.

### quality-gaps.md Never Generated
All 5 themes skipped this document. **For v005:** Either enforce generation or remove from templates. Don't leave it as an ignored expectation.

### Formulaic Use Cases
All 10 v005 backlog items had identical formulaic use case text that added no information. **Fixed:** All 10 items updated with authentic user scenarios during this exploration.

## Tech Debt Addressed in v004

| Item | Status |
|------|--------|
| InMemory test doubles | Resolved (BL-020) |
| Dependency injection for testing | Resolved (BL-021) |
| Fixture factory | Resolved (BL-022) |
| Blackbox REST API tests | Resolved (BL-023) |
| FFmpeg contract tests | Resolved (BL-024) |
| Async scan endpoint | Resolved (BL-025) |
| Security audit | Resolved (BL-026) |
| Performance benchmarks | Resolved (BL-027) |

## Tech Debt Deferred from v004

| Item | Priority | v005 Impact |
|------|----------|-------------|
| Rust coverage at 75% (target 90%) | Medium | Not in v005 scope; track for v006 |
| Scan endpoint blackbox tests | Medium | May affect BL-036 E2E test design |
| get_repository alias duplication | Low | Consolidate if touching those routers |
| No job expiration/cleanup | Low | Jobs remain in memory; acceptable for current scale |
| ResourceWarning from sqlite3 connections | Low | 12 warnings in blackbox tests; not failures |

## Architectural Decisions Informing v005

### Python/Rust Boundary (LRN-011)
Rust handles input sanitization and type safety; Python handles business logic. For v005, this means the thumbnail pipeline (BL-032) should use Rust for FFmpeg command building and Python for orchestration/storage.

### PyO3 FFI Overhead (LRN-012)
Rust is 7-10x slower than Python for simple operations due to FFI overhead. Only use Rust for string-heavy processing (filter generation, command building) where it's 1.9x faster. Avoid Rust for simple data transformations in the GUI layer.

### asyncio.Queue for Job Queue (LRN-010)
The stdlib asyncio.Queue pattern works well for single-worker scenarios. The thumbnail pipeline should integrate with the existing job queue rather than introducing a separate processing mechanism.

### Empty Allowlist = Unrestricted (LRN-017)
Security configuration uses empty allowlist as backwards-compatible default. Maintain this pattern if adding any frontend-related security configuration.
