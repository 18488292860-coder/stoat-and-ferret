# 002 Backlog Analysis — v009

v009 comprises 6 backlog items across 2 themes (observability-pipeline, gui-runtime-fixes), all tagged as wiring gaps from earlier versions. The previous version (v008) achieved 100% first-iteration success with its maintenance-focused wiring work, validating that well-scoped wiring tasks are low-risk. Key retrospective insight: idempotent startup patterns and mypy-as-safety-net apply directly to v009's DI wiring work.

## Backlog Overview

- **Total items:** 6
- **Priority distribution:** 2x P1 (BL-059, BL-063), 4x P2 (BL-057, BL-060, BL-064, BL-065)
- **All items:** open, size L, tagged as wiring gaps
- **Missing items:** None — all 6 items found and complete

## Quality Assessment Summary

| Item | Desc Words | Desc Flagged | AC Flagged | Use Case | Refinement |
|------|-----------|-------------|-----------|----------|------------|
| BL-057 | ~85 | No | 0 | null | Not updatable via API |
| BL-059 | ~75 | No | 0 | null | Not updatable via API |
| BL-060 | ~60 | No | 0 | null | Not updatable via API |
| BL-063 | ~80 | No | 0 | null | Not updatable via API |
| BL-064 | ~70 | No | 0 | null | Not updatable via API |
| BL-065 | ~80 | No | 0 | null | Not updatable via API |

All descriptions follow the current-state/gap/impact structure with adequate depth. All acceptance criteria use testable verb+outcome format. The `use_case` field is null for all 6 items, but `update_backlog_item` does not expose a `use_case` parameter, so this cannot be corrected post-creation.

## Previous Version

- **Version:** v008
- **Status:** Completed (2026-02-21 to 2026-02-22)
- **Retrospective:** `comms/outbox/versions/execution/v008/retrospective.md`
- **Result:** 4/4 features, 100% first-iteration, 0 quality gate failures

## Key Learnings

1. **Idempotent startup patterns** (LRN-040): File logging handler must guard against duplicate registration — same pattern used for structured logging in v008
2. **mypy catches cross-module wiring errors** (LRN-041): Run mypy after every wiring change — it caught invalid kwargs in v008
3. **Group features by modification point** (LRN-042): Theme 1 features all modify DI/startup code; Theme 2 features all modify API/GUI layer
4. **Constructor DI pattern** (LRN-005): Use `create_app()` kwargs, not `dependency_overrides` — relevant to BL-059/060 wiring
5. **Conditional static mount** (LRN-020): SPA fallback (BL-063) must coexist with conditional GUI mounting
6. **SPA E2E navigation workaround** (LRN-023): BL-063 will fix the root cause that required client-side-only navigation in E2E tests

## Tech Debt

### From v008 (carried forward)
- Lifespan function complexity growing — monitor as BL-057 adds more startup logic
- E2E timeout audit still pending — remaining specs need explicit timeouts
- Settings consumption lint not yet automated

### Addressed by v009
- BL-059: ObservableFFmpegExecutor dead code (created v002, never wired)
- BL-060: AuditLogger dead code (created v002, never wired)
- BL-063: SPA routing deferred from v005
- BL-065: WebSocket broadcasts deferred from v005
