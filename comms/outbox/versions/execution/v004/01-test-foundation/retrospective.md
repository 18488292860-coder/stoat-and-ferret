# Theme Retrospective — 01: Test Foundation

## Theme Summary

Theme 01 established the core testing infrastructure for the stoat-and-ferret project. Three features were delivered sequentially, each building on the previous:

1. **InMemory test doubles** — deepcopy isolation, seed helpers, and a deterministic job queue
2. **Dependency injection** — constructor-based DI via `create_app()` params, replacing `dependency_overrides`
3. **Fixture factory** — builder-pattern factory with `build()` and `create_via_api()` output modes

All three features passed quality gates (ruff, mypy, pytest) and met their acceptance criteria. The theme delivered 59 new tests, growing the suite from ~396 to 455 passing tests while maintaining 92.74% coverage.

## Feature Results

| # | Feature | Acceptance | Quality Gates | Tests Added | Status |
|---|---------|------------|---------------|-------------|--------|
| 1 | inmemory-test-doubles | 6/6 PASS | All PASS | 36 | Complete |
| 2 | dependency-injection | 5/5 PASS | All PASS | 5 | Complete |
| 3 | fixture-factory | 6/6 PASS | All PASS | 18 | Complete |

**Totals:** 17/17 acceptance criteria passed. 59 new tests. 0 regressions.

## Key Learnings

### What Went Well

- **Sequential feature chain worked cleanly.** Each feature built directly on the previous without conflicts. Feature 2 consumed Feature 1's InMemory repos; Feature 3 consumed Feature 2's DI pattern.
- **YAGNI discipline was effective.** Two planned requirements were correctly deferred:
  - `executor` and `job_queue` params for `create_app()` — not used anywhere yet
  - `with_text_overlay()` on the fixture factory — no TextOverlay domain model exists
- **Contract/parity tests added real value.** 8 tests verifying InMemory matches SQLite behavior provide ongoing safety for test-double drift.
- **Builder pattern for fixtures.** The dual-output (`build()` for unit, `create_via_api()` for integration) design provides clear test ergonomics without forcing a single testing style.

### Patterns Discovered

- **Deepcopy on both read and write** is the correct isolation strategy for InMemory repositories. Deepcopy only on read would still allow callers to mutate stored state via input references.
- **Synchronous seed() methods** on test doubles (vs async `add()`) reduce test setup friction. They're test-only utilities and don't need async overhead.
- **`app.state` for DI resolution** — dependency functions check `app.state` first, falling back to constructing production instances. This is cleaner than FastAPI's `dependency_overrides` for test wiring.

### What Could Improve

- **Feature 3 had a scope mismatch.** The requirements called for `with_text_overlay()` and incremental migration of existing tests, but neither was achievable given the codebase state. Requirements should validate against the actual domain model before execution.
- **`get_repository` / `get_video_repository` alias consolidation** was noted in the theme design but not fully addressed. The DI feature made both work correctly but didn't eliminate the alias.

## Technical Debt

| Item | Source | Priority | Notes |
|------|--------|----------|-------|
| `get_repository` / `get_video_repository` are aliases for the same video repository | Feature 2 | Low | Both resolve correctly via DI; consolidate when touching those routers |
| Existing tests not migrated to fixture factory | Feature 3 | Low | Factory available for new tests and incremental adoption |
| `with_text_overlay()` not implemented | Feature 3 | Deferred | Add when TextOverlay domain model is created |
| `executor` / `job_queue` not injectable via `create_app()` | Feature 2 | Deferred | Add when job queue integration is needed (Theme 03) |
| Health check returns synthetic OK in injection mode | Feature 2 | Low | Appropriate for tests but means readiness checks don't validate real DB in test mode |

## Recommendations

1. **For Theme 02 (blackbox-test-catalog):** Use the `api_factory` fixture and `create_via_api()` path for all new integration tests. This validates the full HTTP stack and exercises the DI wiring.
2. **For Theme 03 (job-queue-infrastructure):** The `AsyncJobQueue` protocol and `InMemoryJobQueue` are ready. Add `job_queue` parameter to `create_app()` when wiring up the real implementation.
3. **For future theme designs:** Validate acceptance criteria against the actual domain model during design, not execution. FR-3 of the fixture factory was unachievable from the start.
4. **Incremental test migration:** Don't bulk-migrate existing tests to the factory. Migrate them as they are touched for other reasons.

## Metrics

- **Lines changed:** +2,449 / -117 across 31 files
- **Source files changed:** 10 (6 modified, 4 created)
- **Test files changed:** 12 (4 modified, 8 created)
- **Final test count:** 455 passed, 8 skipped
- **Coverage:** 92.74%
- **Commits:** 3 (one per feature, via PR)
