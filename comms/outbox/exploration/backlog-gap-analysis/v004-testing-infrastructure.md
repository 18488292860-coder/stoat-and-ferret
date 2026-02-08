# v004 Gap Analysis: Testing Infrastructure & Quality Verification

**Milestones:** M1.8 (Black Box Testing Infrastructure), M1.9 (Quality Verification)
**Design docs:** 07-quality-architecture.md (primary), 01-roadmap.md

## Existing Coverage

- BL-009: Property test guidance (P2) — partially covers M1.8 proptest aspect
- BL-010: Rust code coverage (P3) — partially covers M1.9 coverage measurement
- BL-012: Coverage reporting gaps (P3) — partially covers M1.9
- BL-014: Docker-based local testing (P2) — testing infrastructure
- BL-016: Unify search behavior (P3) — contract test gap

## Gaps Identified

### M1.8: Black Box Testing Infrastructure

**Gap 1: Recording Test Double Catalog**
The RecordingFFmpegExecutor exists (v002), but InMemoryProjectStorage and InMemoryJobQueue are missing. NullPreviewEngine is not yet needed but specified in 07-quality-architecture.md.

**Gap 2: Application Test Wiring**
`create_app()` exists but does not accept injectable dependencies per 07-quality-architecture.md spec. Test wiring needs dependency injection slots for all test doubles.

**Gap 3: Fixture Factory**
No builder-pattern fixture factory exists. 07-quality-architecture.md specifies `with_clip()`, `with_text_overlay()`, `build()`, and `create_via_api()` methods.

**Gap 4: Black Box Test Scenarios**
No black box tests exist. Spec requires: scan→project→clips workflow, validation error scenarios, WebSocket lifecycle tests.

**Gap 5: Contract Tests for Test Doubles**
Only InMemoryVideoRepository has contract tests. No contract tests verify InMemoryProjectStorage or InMemoryClipRepository match their real implementations.

### M1.9: Quality Verification

**Gap 6: Contract Tests for FFmpeg Commands**
v001 deferred "contract tests with real FFmpeg." M1.9 requires validating generated FFmpeg commands produce correct output.

**Gap 7: Security Review of Path Handling**
M1.9 specifies a security review of Rust sanitization. No audit artifact exists.

**Gap 8: Rust vs Python Benchmark**
M1.9 requires benchmarking Rust core against pure-Python equivalents. No benchmark infrastructure exists.

## Suggested Backlog Items

### 1. Implement InMemory Test Doubles for Projects and Jobs
- **Priority:** P1 | **Tags:** `v004`, `testing`, `test-doubles` | **Size:** M
- **Description:** Create InMemoryProjectStorage and InMemoryJobQueue per 07-quality-architecture.md spec. InMemoryProjectStorage needs deepcopy isolation, seed helper, and list/retrieve. InMemoryJobQueue needs controllable execution and configurable outcomes.
- **Acceptance criteria:**
  1. InMemoryProjectStorage implements AsyncProjectRepository protocol
  2. InMemoryJobQueue provides synchronous deterministic execution
  3. Both have seed helpers for test setup
  4. Contract tests verify behavior matches real implementations
- **Depends on:** None

### 2. Add Dependency Injection to create_app()
- **Priority:** P1 | **Tags:** `v004`, `testing`, `dependency-injection` | **Size:** M
- **Description:** Extend `create_app()` to accept optional dependency overrides (FFmpegExecutor, repositories, job queue). Production uses real implementations when None. Per 07-quality-architecture.md application test wiring spec.
- **Acceptance criteria:**
  1. `create_app()` accepts optional executor, repository, and job queue parameters
  2. Default behavior unchanged (real implementations)
  3. Test mode injects recording/in-memory fakes
  4. At least one integration test demonstrates test wiring
- **Depends on:** Item 1

### 3. Build Fixture Factory with Builder Pattern
- **Priority:** P1 | **Tags:** `v004`, `testing`, `fixtures` | **Size:** M
- **Description:** Implement builder-pattern fixture factory per 07-quality-architecture.md. Provides `with_clip()`, `with_text_overlay()`, `build()` (direct), and `create_via_api()` (HTTP) methods.
- **Acceptance criteria:**
  1. Builder creates test projects with configurable clips and effects
  2. `build()` returns objects directly for unit tests
  3. `create_via_api()` exercises full HTTP path for black box tests
  4. Pytest fixtures use the factory for consistent test data
- **Depends on:** Item 2

### 4. Implement Black Box Test Scenario Catalog
- **Priority:** P1 | **Tags:** `v004`, `testing`, `black-box` | **Size:** L
- **Description:** Create black box integration tests per 07-quality-architecture.md. Tests exercise complete workflows through REST API using real Rust core + recording fakes.
- **Acceptance criteria:**
  1. Core workflow test: scan → project → clips (effects/render deferred to v006+)
  2. Error handling tests: validation errors, FFmpeg failures
  3. All tests use recording test doubles, never mock Rust core
  4. Tests run in CI without FFmpeg installed
  5. pytest markers separate black box tests from unit tests
- **Depends on:** Items 1–3
- **Note:** WebSocket tests should be included if WebSocket endpoint exists by v004; otherwise defer.

### 5. Contract Tests with Real FFmpeg
- **Priority:** P2 | **Tags:** `v004`, `testing`, `ffmpeg`, `contract` | **Size:** M
- **Description:** Validate that RecordingFFmpegExecutor and FakeFFmpegExecutor produce identical behavior to RealFFmpegExecutor for a representative set of commands. Deferred from v001.
- **Acceptance criteria:**
  1. Parametrized tests run against Real, Recording, and Fake executors
  2. At least 5 representative FFmpeg commands tested
  3. Tests marked with `@pytest.mark.requires_ffmpeg` for CI skip
  4. Contract violations fail the test suite
- **Depends on:** None

### 6. Security Audit of Rust Sanitization
- **Priority:** P2 | **Tags:** `v004`, `security`, `audit` | **Size:** S
- **Description:** Conduct and document security review of Rust path validation and input sanitization per M1.9. Produce audit artifact documenting coverage and any gaps.
- **Acceptance criteria:**
  1. Review covers path traversal, null bytes, shell injection vectors
  2. Audit document in docs/ with findings
  3. Any gaps addressed with new tests or code fixes
- **Depends on:** None
- **Note:** May need EXP before implementation if scope is unclear.

### 7. Rust vs Python Performance Benchmark
- **Priority:** P3 | **Tags:** `v004`, `benchmarking`, `rust` | **Size:** S
- **Description:** Benchmark Rust core operations against pure-Python equivalents per M1.9. Focus on timeline math and filter generation as representative workloads.
- **Acceptance criteria:**
  1. Benchmark script comparing Rust vs Python for 3+ operations
  2. Results documented with speedup ratios
  3. Benchmark runnable via `uv run python benchmarks/...`
- **Depends on:** None

### 8. Async Job Queue for Scan Operations
- **Priority:** P2 | **Tags:** `v004`, `async`, `scan` | **Size:** M
- **Description:** Replace blocking scan endpoint with async job queue. Identified as tech debt in v003 retrospective. Required for black box test scenarios involving scan progress.
- **Acceptance criteria:**
  1. Scan endpoint returns job ID immediately
  2. Job status queryable via GET endpoint
  3. InMemoryJobQueue supports synchronous test execution
  4. Existing scan tests updated
- **Depends on:** Item 1 (InMemoryJobQueue)
