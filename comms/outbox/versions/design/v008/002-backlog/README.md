# Task 002: Backlog Analysis — v008

v008 scope covers 4 backlog items (2x P0, 1x P1, 1x P2) addressing startup wiring gaps and CI stability. All items are well-defined wiring fixes with existing infrastructure — no greenfield design needed. The v007 retrospective highlights that the flaky E2E test (BL-055) blocked multiple PR merges, making its fix a clear priority. LRN-033 reinforces this: fix CI reliability before dependent development cycles.

## Backlog Overview

- **Total items:** 4 (all mandatory per PLAN.md)
- **Priority distribution:** P0: 2 (BL-055, BL-058), P1: 1 (BL-056), P2: 1 (BL-062)
- **Size distribution:** L: 3 (BL-055, BL-058, BL-062), XL: 1 (BL-056)
- **All items status:** open
- **Missing items:** None — all 4 items fetched successfully

## Previous Version

- **Version:** v007 (Effect Workshop GUI)
- **Status:** Completed 2026-02-19
- **Themes:** 4/4 completed, 11/11 features, 131/131 acceptance criteria
- **Retrospective:** `comms/outbox/versions/execution/v007/retrospective.md`
- **Published summary:** `docs/versions/v007/VERSION_SUMMARY.md`

## Key Learnings Applicable to v008

1. **LRN-033 — Fix CI Reliability First:** Directly motivates BL-055 as P0. The flaky E2E test blocked multiple v007 PR merges across themes 03 and 04.
2. **LRN-031 — Detailed Design Specs = First-Iteration Success:** All 4 items have clear current-state/gap/impact descriptions; this positions v008 for clean execution.
3. **LRN-016 — Validate AC Against Codebase:** ACs reference specific functions and files (`configure_logging()`, `create_tables()`, `settings.debug`, `ws.py`). Validate these exist before design.
4. **LRN-019 — Infrastructure First:** Database startup (BL-058) should precede logging (BL-056), as logging may emit during DB initialization.
5. **LRN-029 — Conscious Simplicity:** BL-058's "Alembic or create_tables" AC allows choosing the simpler path with a documented upgrade trigger.

## Tech Debt

From v007 retrospective, relevant to v008:
- **BL-055 (P0, High):** Flaky E2E test — directly addressed in v008
- **SPA fallback (BL-063):** Scheduled for v009, not v008
- **Large definitions.py:** Not relevant to v008 scope
- **Stub file duplication:** Not relevant to v008 scope

## Quality Assessment Summary

| ID | Title | Words | Desc OK | AC Testable | Use Case |
|----|-------|-------|---------|-------------|----------|
| BL-055 | Fix flaky E2E test | ~91 | Yes | 3/3 | Missing |
| BL-056 | Wire structured logging | ~108 | Yes | 6/6 | Authentic |
| BL-058 | Wire database schema creation | ~67 | Yes | 4/4 | Missing |
| BL-062 | Wire orphaned settings | ~74 | Yes | 3/3 | Missing |

Notes: BL-055, BL-058, BL-062 have null use_case fields. The `update_backlog_item` API does not expose a `use_case` parameter, so these cannot be updated programmatically. BL-056 has an authentic, specific use case.
