# Learnings Summary — Applicable to v005

## Highly Relevant

### LRN-019: Build Infrastructure First in Sequential Version Planning
- **Summary:** Sequence infrastructure themes before consumer themes; the upfront investment pays off when subsequent themes consume the infrastructure.
- **Applicability:** Direct. v005 must scaffold the frontend (BL-028) and WebSocket (BL-029) before building GUI panels (BL-030, BL-031, BL-033, BL-035). Theme sequencing is critical.

### LRN-016: Validate Acceptance Criteria Against Codebase During Design
- **Summary:** Validate that AC references existing domain models and APIs during design to prevent unachievable requirements during execution.
- **Applicability:** Direct. GUI items reference backend APIs (/health/ready, /api/videos/search, /api/projects, /ws). Verify these endpoints exist and match the expected signatures before finalizing AC.

### LRN-005: Constructor DI over dependency_overrides for FastAPI Testing
- **Summary:** Use `create_app()` constructor parameters instead of `dependency_overrides` for cleaner test wiring.
- **Applicability:** Direct. WebSocket manager (BL-029) and thumbnail service (BL-032) should be injected via `create_app()` kwargs, consistent with the established pattern.

### LRN-008: Record-Replay with Strict Mode for External Dependency Testing
- **Summary:** Three-tier executor pattern (Real/Recording/Fake) with strict argument verification enables reliable testing.
- **Applicability:** Direct. BL-032 (thumbnail pipeline) should use RecordingFFmpegExecutor for thumbnail generation command testing, following the established pattern.

### LRN-011: Python Business Logic, Rust Input Safety in Hybrid Architectures
- **Summary:** Rust handles input sanitization and type safety; Python handles business policy and domain rules.
- **Applicability:** Direct. Thumbnail FFmpeg commands should be built in Rust; orchestration/storage logic stays in Python.

## Moderately Relevant

### LRN-012: PyO3 FFI Overhead Makes Rust Slower for Simple Operations
- **Summary:** PyO3 FFI overhead makes Rust 7-10x slower for simple operations; Rust wins only for string-heavy processing (1.9x faster).
- **Applicability:** Moderate. Avoid routing simple GUI data transformations through Rust. Use Rust only for filter/command generation and timeline math where it adds value.

### LRN-009: Handler Registration Pattern for Generic Job Queues
- **Summary:** Explicit handler registration with factory-based DI provides clean, testable job routing.
- **Applicability:** Moderate. Thumbnail generation could be a new job type registered with the existing job queue, reusing the handler registration pattern.

### LRN-010: Prefer stdlib asyncio.Queue over External Queue Dependencies
- **Summary:** stdlib asyncio.Queue with producer-consumer pattern eliminates external dependencies.
- **Applicability:** Moderate. Thumbnail pipeline should integrate with existing AsyncioJobQueue rather than introducing new dependencies.

### LRN-004: Deepcopy Isolation for InMemory Test Doubles
- **Summary:** InMemory test doubles must deepcopy on read and write to prevent mutation leakage.
- **Applicability:** Moderate. Any new InMemory doubles for v005 services (WebSocket manager mock, thumbnail service mock) should follow this pattern.

### LRN-007: Parity Tests Prevent Test Double Drift
- **Summary:** Parity tests running identical operations against fakes and real implementations catch behavioral drift.
- **Applicability:** Moderate. If creating InMemory doubles for new v005 services, add parity tests.

## Background Reference

### LRN-015: Single-Matrix CI Jobs for Expensive Operations
- **Summary:** Run expensive CI operations on a single platform rather than the full matrix.
- **Applicability:** Low-moderate. Playwright E2E tests (BL-036) may be expensive; consider running on ubuntu-latest only.

### LRN-013: Progressive Coverage Thresholds for Bootstrapping Enforcement
- **Summary:** Start with conservative thresholds and ratchet up after confirming baseline.
- **Applicability:** Low-moderate. Frontend test coverage thresholds should start conservative.

### LRN-017: Empty Allowlist Means Unrestricted as Backwards-Compatible Default
- **Summary:** Empty allowlist defaults to unrestricted for backwards compatibility.
- **Applicability:** Low. Maintain pattern if adding frontend security configuration.

### LRN-014: Template-Driven Process Improvements
- **Summary:** Embed process improvements in templates rather than standalone documentation.
- **Applicability:** Low. Frontend conventions should be embedded in component templates.

### LRN-006: Builder Pattern with Dual Output Modes for Test Fixtures
- **Summary:** Fixture factories with build() and create_via_api() modes provide flexibility.
- **Applicability:** Low. Extend existing factories if new domain models are added.
