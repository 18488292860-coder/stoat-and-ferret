# Feature: flaky-e2e-fix

## Goal

Fix the intermittent toBeHidden() timeout in project-creation.spec.ts so E2E tests pass reliably in CI.

## Background

**Backlog Item:** BL-055 (P0)

The E2E test at `gui/e2e/project-creation.spec.ts:31` intermittently fails with a toBeHidden assertion timeout on the project creation modal. The test fails on main (GitHub Actions run 22188785818) independent of any feature branch changes. During v007 execution, this caused the dynamic-parameter-forms feature to receive a "partial" completion status despite 12/12 acceptance criteria passing. The execution pipeline halted, requiring manual restart.

Root cause: The default Playwright assertion timeout is 5000ms (5 seconds), which is insufficient for GitHub Actions CI environments where API response + React re-render + DOM update can exceed 5 seconds. No modal animation exists — the race is between API response time and DOM state update.

## Functional Requirements

**FR-001:** toBeHidden assertion has explicit timeout
- Acceptance: project-creation.spec.ts:31 toBeHidden assertion passes reliably across 10 consecutive CI runs

**FR-002:** No retry loops required
- Acceptance: No E2E test requires retry loops to pass in CI

**FR-003:** Functionality preserved
- Acceptance: Flaky test fix does not alter the tested project creation functionality (timeout parameter only)

## Non-Functional Requirements

**NFR-001:** Timeout value follows project conventions
- Metric: Explicit timeout uses 10,000ms matching established project pattern (scan.spec.ts uses 10000)

## Handler Pattern

Not applicable for v008 — no new handlers introduced.

## Out of Scope

- Global Playwright timeout configuration changes
- Other E2E test modifications
- Project creation functionality changes
- CI configuration changes

## Test Requirements

**E2E validation:**
- Verify toBeHidden() assertion passes with explicit { timeout: 10_000 }
- AC requires 10 consecutive CI runs passing — this is a post-merge validation criterion
- Verify no other E2E tests require retry loops
- Verify tested functionality unchanged (modal still opens and closes correctly)

## Reference

See `comms/outbox/versions/design/v008/004-research/` for supporting evidence.
