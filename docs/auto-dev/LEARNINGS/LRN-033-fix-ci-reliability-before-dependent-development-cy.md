## Context

When planning a version or theme that builds on existing CI infrastructure (especially E2E tests), and there are known flaky or failing tests in the CI pipeline.

## Insight

Fix CI reliability issues before starting development cycles that depend on CI for PR merges. A single flaky test can block PR merges across multiple features and themes, creating compounding friction. The cost of fixing CI upfront is far less than the cumulative cost of repeated investigation, retry attempts, and merge delays across multiple features.

## Evidence

In v007, a pre-existing flaky E2E test (BL-055, `project-creation.spec.ts`) blocked PR merges for Theme 03 features 002 and 003, and was flagged again in Theme 04. Three CI retry attempts per feature, plus timeout increase investigation, yielded no resolution. The same test failed consistently on the `main` branch, confirming it was pre-existing. Two themes of development were affected by a single unresolved CI failure.

## Application

- Before starting GUI-heavy or E2E-dependent development, audit CI pipeline health
- Fix or skip-mark known flaky tests before beginning new feature work
- Establish a CI health gate as a precondition for starting development versions
- Track flaky tests as high-priority backlog items, not low-priority tech debt