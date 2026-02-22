# Learnings Summary — v009 Applicability

## Directly Applicable

### LRN-040: Idempotent Startup Functions for Lifespan Wiring
**Summary:** Startup/lifespan functions must be idempotent using operation-appropriate guards to prevent side-effect accumulation in tests and production.
**Applicability:** BL-057 (file logging) extends `configure_logging()` — the RotatingFileHandler must guard against duplicate registration using the same pattern as the stdout handler. BL-059/060 add DI instantiation that must be safe under repeated lifespan invocations in tests.

### LRN-041: Static Type Checking Catches Cross-Module Wiring Errors
**Summary:** mypy catches invalid API calls during cross-module wiring work before runtime.
**Applicability:** All 6 items are cross-module wiring. Run mypy after every change — it's the primary safety net for catching kwarg mismatches, wrong types, and API misuse during wiring.

### LRN-005: Constructor DI over dependency_overrides for FastAPI Testing
**Summary:** Use `create_app()` constructor parameters instead of `dependency_overrides` for cleaner test wiring.
**Applicability:** BL-059 (ObservableFFmpegExecutor) and BL-060 (AuditLogger) must wire through `create_app()` kwargs stored on `app.state`. Tests inject via the same mechanism.

### LRN-020: Conditional Static Mount Pattern for Optional Frontend Serving
**Summary:** Only mount StaticFiles when the directory exists to decouple backend testing from frontend build state.
**Applicability:** BL-063 (SPA routing fallback) must coexist with this pattern — the fallback route should only activate when the GUI directory exists.

### LRN-023: Client-Side Navigation for SPA E2E Tests Without Server-Side Routing
**Summary:** SPA E2E tests must use client-side navigation from root path when the server doesn't handle SPA fallback.
**Applicability:** BL-063 fixes the root cause documented in this learning. Once SPA fallback works, E2E tests can navigate directly to sub-paths, simplifying test setup.

### LRN-042: Group Features by Modification Point for Theme Cohesion
**Summary:** Group features that modify the same code path into a single theme.
**Applicability:** v009 themes already follow this — Theme 1 groups DI/startup modifications, Theme 2 groups API/GUI layer modifications.

### LRN-043: Explicit Assertion Timeouts in CI-Bound E2E Tests
**Summary:** Add explicit timeouts to all state-transition assertions in E2E specs.
**Applicability:** Any new E2E tests for BL-063 (SPA routing) or BL-065 (WebSocket broadcasts) must include explicit timeouts to avoid the flake pattern fixed in v008.

## Process Guidance

### LRN-046: Maintenance Versions Succeed with Well-Understood Change Scoping
**Summary:** Maintenance-focused versions with well-understood changes achieve higher first-iteration success.
**Applicability:** v009 is a maintenance/wiring version like v008. Expect high first-iteration success if scope stays tight.

### LRN-031: Detailed Design Specifications Correlate with First-Iteration Success
**Summary:** Detailed design specs with specific AC and implementation plans lead to first-iteration success.
**Applicability:** All 6 items have detailed descriptions with current-state/gap/impact structure and testable ACs. Design phase should add implementation plans with specific code paths.

### LRN-016: Validate Acceptance Criteria Against Codebase During Design
**Summary:** Validate that ACs reference existing domain models and APIs during design.
**Applicability:** During theme design, verify that referenced classes (ObservableFFmpegExecutor, AuditLogger, AsyncProjectRepository, ConnectionManager) exist and have the expected interfaces.

## Tangentially Relevant

### LRN-009: Handler Registration Pattern for Generic Job Queues
**Summary:** Explicit handler registration with factory-based DI provides clean, testable job routing.
**Applicability:** Reference pattern for DI wiring approach, though v009 uses constructor DI rather than handler registration.

### LRN-037: Feature Composition Succeeds Through Independent Store Interfaces
**Summary:** Design each feature as a standalone component with an independent state store.
**Applicability:** BL-065 (WebSocket broadcasts) touches the frontend ActivityLog — any changes should respect the independent store pattern used for Zustand stores.

### LRN-044: Settings Consumer Traceability as a Completeness Check
**Summary:** Verify every settings field has a production consumer after wiring work.
**Applicability:** BL-057 may add new settings (backup count). Any new settings fields must be wired to consumers immediately.
