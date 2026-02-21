# Theme: ci-stability

## Goal

Eliminate the flaky E2E test that intermittently blocks CI merges, restoring reliable CI signal for all future development. During v007 execution, this flake caused a false-positive halt on the dynamic-parameter-forms feature despite all acceptance criteria passing.

## Design Artifacts

See `comms/outbox/versions/design/v008/006-critical-thinking/` for full risk analysis.

## Features

| # | Feature | Backlog | Goal |
|---|---------|---------|------|
| 001 | flaky-e2e-fix | BL-055 | Fix toBeHidden() timeout in project-creation.spec.ts so E2E passes reliably |

## Dependencies

- No dependencies on Theme 1. This theme is fully independent and touches only frontend test infrastructure.

## Technical Approach

- Add explicit `{ timeout: 10_000 }` to the `toBeHidden()` assertion at `gui/e2e/project-creation.spec.ts:37`, matching established project patterns (`scan.spec.ts` uses 10000, `accessibility.spec.ts` uses 15_000) (see `004-research/evidence-log.md` for timeout value evidence)
- Root cause: default 5s Playwright assertion timeout is too short for GitHub Actions CI environment

## Risks

| Risk | Mitigation |
|------|------------|
| "10 consecutive CI runs" AC not automatable in single PR | Accept fix based on root cause analysis; validate post-merge; see `006-critical-thinking/risk-assessment.md` |
