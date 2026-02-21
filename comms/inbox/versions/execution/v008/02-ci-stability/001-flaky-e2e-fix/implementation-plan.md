# Implementation Plan: flaky-e2e-fix

## Overview

Add an explicit `{ timeout: 10_000 }` parameter to the `toBeHidden()` assertion in `project-creation.spec.ts` to eliminate the intermittent timeout failure on GitHub Actions CI. The fix matches the project's established timeout pattern.

## Files to Create/Modify

| File | Action | Change |
|------|--------|--------|
| `gui/e2e/project-creation.spec.ts` | Modify | Add `{ timeout: 10_000 }` to `toBeHidden()` assertion at line 37 |

## Test Files

`gui/e2e/project-creation.spec.ts`

## Implementation Stages

### Stage 1: Add explicit timeout

1. In `gui/e2e/project-creation.spec.ts`, locate the `toBeHidden()` assertion (line 37)
2. Add explicit timeout parameter: `.toBeHidden({ timeout: 10_000 })`
3. Verify the fix matches the project's established pattern (scan.spec.ts uses 10000, accessibility.spec.ts uses 15_000)

**Verification:**
```bash
cd gui && npx playwright test project-creation.spec.ts --reporter=list
```

### Stage 2: Verify other E2E tests unaffected

1. Run the full E2E test suite to ensure no regressions
2. Confirm no other E2E tests require retry loops

**Verification:**
```bash
cd gui && npx playwright test --reporter=list
```

## Test Infrastructure Updates

- No new test files or fixtures
- No conftest or playwright.config.ts changes

## Quality Gates

```bash
cd gui
npx tsc -b
npx playwright test --reporter=list
```

## Risks

- "10 consecutive CI runs" AC is a post-merge validation criterion — see `006-critical-thinking/risk-assessment.md`
- If flake reappears post-merge, create follow-up backlog item

## Commit Message

```
fix: add explicit timeout to flaky E2E toBeHidden assertion (BL-055)
```