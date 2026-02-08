# Refined Logical Design — v004: Testing Infrastructure & Quality Verification

## Version Overview

**Version**: v004
**Goal**: Build comprehensive testing and quality verification infrastructure covering black box testing, recording test doubles, contract tests, and quality verification suite (milestones M1.8–M1.9).
**Scope**: 13 backlog items (4 P1, 5 P2, 4 P3) organized into 5 themes with 15 features.
**Baseline**: v003 completed with 395 tests, 93% Python coverage, 4 themes, 15 features.
**Estimated new tests**: 85–130 (target ~500 total).

**Risk review**: All 8 risks and 5 unknowns investigated (Task 006). No structural changes to theme groupings or feature ordering. 4 risks downgraded in severity. No blockers identified. See risk-assessment.md for full details.

## Theme Breakdown

### Theme 01: Test Foundation (`01-test-foundation`)

**Goal**: Establish core testing infrastructure — test doubles, dependency injection, and fixture factories.

| # | Feature | Backlog | Goal | Dependencies |
|---|---------|---------|------|-------------|
| 1 | `010-inmemory-test-doubles` | BL-020 | Add deepcopy isolation and seed helpers to AsyncInMemoryProjectRepository; create InMemoryJobQueue | None |
| 2 | `020-dependency-injection` | BL-021 | Add optional injectable parameters to create_app() replacing dependency_overrides | 010 |
| 3 | `030-fixture-factory` | BL-022 | Build builder-pattern fixture factory with build() and create_via_api() methods | 020 |

**Refinements from risk review**:
- R2 resolved: `copy.deepcopy()` is fine for flat scalar dataclasses (`Project`: 7 fields, `Video`: 14 fields). No performance optimization needed.
- R5 confirmed: 4 `dependency_overrides` in conftest.py at known locations (lines 129-132). Note `get_repository` and `get_video_repository` both alias the same instance — DI migration should consolidate.
- `create_app()` currently takes zero parameters (app.py:43). Migration adds optional params defaulting to `None`.

### Theme 02: Black Box & Contract Testing (`02-blackbox-contract`)

**Goal**: Implement end-to-end black box tests, verified fakes for FFmpeg executors, and unified search behavior.

| # | Feature | Backlog | Goal | Dependencies |
|---|---------|---------|------|-------------|
| 1 | `010-blackbox-test-catalog` | BL-023 | Core workflow and error handling tests through REST API using test doubles | Theme 01 complete |
| 2 | `020-ffmpeg-contract-tests` | BL-024 | Parametrized tests across Real, Recording, and Fake executors with args verification | None (after Theme 01) |
| 3 | `030-search-unification` | BL-016 | Align InMemory search with FTS5 prefix-match semantics | None |

**Refinements from risk review**:
- U3 resolved: FakeFFmpegExecutor records args but ignores them during replay (executor.py:198-236). Add optional `strict` parameter to verify args match recording. Contract tests use strict mode.
- R3 resolved: FTS5 uses quoted phrase prefix match (`'"{query}"*'`). InMemory uses substring (`in`). Three known gaps: (1) substring vs prefix, (2) multi-word phrase handling, (3) field scope. Per-token `startswith` closes gap #1; gaps #2–#3 are minor for current usage patterns.

### Theme 03: Async Scan Infrastructure (`03-async-scan`)

**Goal**: Replace synchronous blocking scan with async job queue pattern.

| # | Feature | Backlog | Goal | Dependencies |
|---|---------|---------|------|-------------|
| 1 | `010-job-queue-infrastructure` | BL-027 (part 1) | AsyncJobQueue protocol, asyncio.Queue implementation, lifespan integration | Theme 01 feature 010 |
| 2 | `020-async-scan-endpoint` | BL-027 (part 2) | Refactor scan endpoint to return job ID, add GET /jobs/{id} endpoint | 010 |
| 3 | `030-scan-doc-updates` | BL-027 (part 3) | Update API spec and design docs for async scan | 020 |

**Refinements from risk review**:
- R1 resolved: **No external consumers** of the scan endpoint beyond the test suite. Breaking change is safe — only test migration needed. Severity downgraded from High to Low.
- U1 unchanged: Job queue timeout values require runtime testing. Default 5-minute timeout with per-job configurability.

### Theme 04: Security & Performance Verification (`04-security-performance`)

**Goal**: Complete M1.9 quality verification — security audit and performance benchmarks.

| # | Feature | Backlog | Goal | Dependencies |
|---|---------|---------|------|-------------|
| 1 | `010-security-audit` | BL-025 | Audit Rust path validation, input sanitization, and escape_filter_text | None |
| 2 | `020-performance-benchmark` | BL-026 | Benchmark Rust vs Python for 3+ operations with documented speedup ratios | None |

**Refinements from risk review**:
- R4 resolved: `validate_path` (sanitize/mod.rs:230-238) only checks empty/null. Python `scan_directory` has no directory allowlist. Gap is real but scope is bounded: recommend adding `ALLOWED_SCAN_ROOTS` config + check in scan service (~20 lines). No Rust changes needed — deferred-to-Python architecture is correct.
- Scope creep risk LOW: Audit produces findings document, critical fix is small and predictable.
- U2 unchanged: Benchmark operations selected at implementation time.

### Theme 05: Developer Experience & Coverage (`05-devex-coverage`)

**Goal**: Improve developer tooling, testing processes, and coverage infrastructure.

| # | Feature | Backlog | Goal | Dependencies |
|---|---------|---------|------|-------------|
| 1 | `010-property-test-guidance` | BL-009 | Add property test section to feature design templates | None |
| 2 | `020-rust-coverage` | BL-010 | Configure llvm-cov with CI integration and threshold enforcement | None |
| 3 | `030-coverage-gap-fixes` | BL-012 | Review and fix coverage exclusions for ImportError fallback code | None |
| 4 | `040-docker-testing` | BL-014 | Create Dockerfile and docker-compose.yml for containerized testing | None |

**Refinements from risk review**:
- R6 resolved: ~160+ Rust tests across 12 files. Baseline likely 75-90%. If below 90%, set progressive threshold.
- U5 resolved: llvm-cov runs on single CI matrix combo (ubuntu-latest + Python 3.12), uploads lcov as artifact.
- R7/R8 unchanged: Acceptable low risks with clear mitigations.

## Execution Order

No changes from Task 005. Risk investigation confirmed all dependencies and ordering are correct.

### Phase 1: Foundation (Theme 01) — strictly sequential
1. `01-test-foundation/010-inmemory-test-doubles` (BL-020)
2. `01-test-foundation/020-dependency-injection` (BL-021)
3. `01-test-foundation/030-fixture-factory` (BL-022)

### Phase 2: Consumers & Independent Work (Themes 02–05)
4. `02-blackbox-contract/010-blackbox-test-catalog` (BL-023)
5. `03-async-scan/010-job-queue-infrastructure` (BL-027 part 1)
6. `03-async-scan/020-async-scan-endpoint` (BL-027 part 2)
7. `03-async-scan/030-scan-doc-updates` (BL-027 part 3)
8. `02-blackbox-contract/020-ffmpeg-contract-tests` (BL-024)
9. `02-blackbox-contract/030-search-unification` (BL-016)
10. `04-security-performance/010-security-audit` (BL-025)
11. `04-security-performance/020-performance-benchmark` (BL-026)
12. `05-devex-coverage/010-property-test-guidance` (BL-009)
13. `05-devex-coverage/020-rust-coverage` (BL-010)
14. `05-devex-coverage/030-coverage-gap-fixes` (BL-012)
15. `05-devex-coverage/040-docker-testing` (BL-014)

## Backlog Coverage Verification

| Backlog ID | Priority | Theme | Feature | Status |
|------------|----------|-------|---------|--------|
| BL-020 | P1 | 01-test-foundation | 010-inmemory-test-doubles | Mapped |
| BL-021 | P1 | 01-test-foundation | 020-dependency-injection | Mapped |
| BL-022 | P1 | 01-test-foundation | 030-fixture-factory | Mapped |
| BL-023 | P1 | 02-blackbox-contract | 010-blackbox-test-catalog | Mapped |
| BL-024 | P2 | 02-blackbox-contract | 020-ffmpeg-contract-tests | Mapped |
| BL-025 | P2 | 04-security-performance | 010-security-audit | Mapped |
| BL-026 | P3 | 04-security-performance | 020-performance-benchmark | Mapped |
| BL-027 | P2 | 03-async-scan | 010/020/030 (3 features) | Mapped |
| BL-009 | P2 | 05-devex-coverage | 010-property-test-guidance | Mapped |
| BL-010 | P3 | 05-devex-coverage | 020-rust-coverage | Mapped |
| BL-012 | P3 | 05-devex-coverage | 030-coverage-gap-fixes | Mapped |
| BL-014 | P2 | 05-devex-coverage | 040-docker-testing | Mapped |
| BL-016 | P3 | 02-blackbox-contract | 030-search-unification | Mapped |

All 13 backlog items mapped. No deferrals. No descoping.

## Test Strategy Updates

No structural changes to the test strategy from Task 005. Risk investigation confirmed all test requirements are appropriate. Minor refinements:

1. **BL-024 contract tests**: FakeFFmpegExecutor `strict` mode is the mechanism for args verification (see U3 resolution).
2. **BL-016 parity tests**: Three specific behavioral differences to test: (a) prefix vs substring, (b) multi-word phrase handling, (c) field scope.
3. **BL-010 Rust coverage**: llvm-cov on single CI matrix combo. Progressive threshold if baseline < 90%.
4. **BL-025 security tests**: Path traversal test cases should include `ALLOWED_SCAN_ROOTS` validation after the audit fix.
