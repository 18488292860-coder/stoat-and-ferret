# Theme Retrospective — 05: Developer Experience & Coverage

## Theme Summary

Theme 05 improved developer tooling, testing processes, and coverage infrastructure across four independent features. The theme introduced property-based testing guidance with Hypothesis, added Rust code coverage enforcement via `cargo-llvm-cov` in CI, audited and fixed all coverage exclusions in the Python codebase, and created a Docker-based testing environment for developers facing host restrictions.

All four features passed every quality gate (ruff, mypy, pytest) and met all acceptance criteria. The theme added 7 new tests (4 property test examples + 3 coverage fallback tests), bringing the suite to 571 passed / 15 skipped at 92.86% Python coverage, and established Rust coverage tracking at a 75% threshold.

## Feature Results

| # | Feature | Acceptance | Quality Gates | Key Deliverable | Status |
|---|---------|------------|---------------|-----------------|--------|
| 1 | property-test-guidance | 6/6 PASS | All PASS | Hypothesis dependency + design template updates | Complete |
| 2 | rust-coverage | 4/4 PASS | All PASS | `cargo-llvm-cov` CI job with 75% threshold | Complete |
| 3 | coverage-gap-fixes | 4/4 PASS | All PASS | Audit of all exclusions + fallback tests | Complete |
| 4 | docker-testing | 3/3 PASS | All PASS | Multi-stage Dockerfile + docker-compose | Complete |

**Totals:** 17/17 acceptance criteria passed. 7 new tests. 0 regressions.

## Key Learnings

### What Went Well

- **All features were fully independent.** Zero inter-feature dependencies meant each could be implemented and merged in any order without blocking. This is the cleanest theme structure in v004 so far.
- **Progressive thresholds avoid false failures.** Setting Rust coverage to 75% initially (with a documented target of 90%) prevents CI failures while the baseline is established. The threshold can be ratcheted up once actual coverage is confirmed — a practical approach to bootstrapping enforcement.
- **Coverage audit revealed only one genuine gap.** The `pragma: no cover` on the ImportError fallback in `__init__.py` was the only unjustified exclusion. All 15 `type: ignore` comments and the `noqa` in the stub file were properly justified. The codebase was already in good shape.
- **Docker multi-stage build cleanly separates concerns.** Builder stage (Rust compilation) vs runtime stage (Python only) keeps the final image small while caching the expensive Rust build layer. Bind mounts for Python source mean only Rust/dependency changes require rebuilds.
- **Invariant-first design template is actionable.** The property test guidance integrates into the existing requirements/implementation-plan templates with a concrete ID format (PT-xxx) and expected test count tracking, making it easy for future features to adopt.

### Patterns Discovered

- **Template-driven process improvements scale well.** Rather than writing standalone documentation, embedding property test guidance directly into the feature design templates (02-REQUIREMENTS.md, 03-IMPLEMENTATION-PLAN.md) ensures it becomes part of the standard workflow for all future features.
- **Single-matrix CI jobs for expensive operations.** Running `cargo-llvm-cov` only on ubuntu-latest (not across the full OS/Python matrix) keeps CI fast while still enforcing coverage. This pattern should be used for any future expensive CI steps.
- **Audit-then-fix is more efficient than blanket changes.** Feature 003 first catalogued all exclusions before deciding which needed action. This prevented unnecessary churn — only 1 of 21 exclusions needed remediation.

### What Could Improve

- **No quality-gaps documents were generated.** This is the fourth consecutive theme without `quality-gaps.md` files. The completion reports continue to serve as the sole quality record. The prompt template should either enforce quality-gaps generation or remove it from the expected outputs.
- **Rust coverage threshold needs follow-up.** The 75% threshold is a placeholder. Once CI confirms the actual baseline, it should be ratcheted to 90% per AGENTS.md standards. This requires a manual follow-up action.
- **Docker testing is not CI-integrated.** The Docker environment is for local developer use only. If containerized testing is needed in CI (e.g., for integration tests), additional workflow changes would be required.

## Technical Debt

| Item | Source | Priority | Notes |
|------|--------|----------|-------|
| Rust coverage threshold at 75%, target is 90% | Feature 2 | Medium | Ratchet `--fail-under-lines` once CI baseline is confirmed |
| Docker image not used in CI | Feature 4 | Low | Currently local-only; could be used for integration test isolation if needed |
| Property test examples are illustrative only | Feature 1 | Low | Example tests in `tests/examples/` use domain objects but don't test real business logic; real property tests should be added as features are developed |
| No quality-gaps.md generation in pipeline | All | Low | Fourth theme without these documents; formalize as optional or enforce generation |

## Recommendations

1. **Ratchet Rust coverage:** After 2-3 CI runs confirm the baseline, update `.github/workflows/ci.yml` to increase `--fail-under-lines` toward 90%. Track the actual coverage number from CI artifacts.
2. **Adopt property tests in next version:** The templates and Hypothesis dependency are in place. Future features with domain invariants (e.g., timeline arithmetic, clip constraints) should include PT-xxx criteria in their requirements.
3. **Resolve quality-gaps process gap:** Either add quality-gaps generation as a required step in feature execution prompts, or remove references to `quality-gaps.md` from retrospective templates and completion criteria.
4. **Docker for onboarding:** The Docker environment lowers the barrier for new contributors who may not have Rust or maturin installed. Consider mentioning it prominently in the main README's development setup section.

## Metrics

- **Lines changed:** +1,239 / -13 across 27 files
- **Source files modified:** 1 (`stoat_ferret_core/__init__.py`)
- **CI files modified:** 1 (`.github/workflows/ci.yml`)
- **Template/docs files modified:** 4
- **Test files created:** 2 (`test_property_example.py`, `test_import_fallback.py`)
- **Infrastructure files created:** 3 (`Dockerfile`, `docker-compose.yml`, `.dockerignore`)
- **Final test count:** 571 passed, 15 skipped
- **Python coverage:** 92.86%
- **Rust coverage threshold:** 75% (target: 90%)
- **Commits:** 4 (one per feature, via PRs #59–#62)
