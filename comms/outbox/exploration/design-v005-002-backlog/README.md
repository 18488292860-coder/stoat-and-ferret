# Exploration: design-v005-002-backlog

v005 encompasses 10 backlog items delivering the first GUI layer for stoat-and-ferret: frontend scaffolding, WebSocket real-time events, application shell, dashboard, thumbnail pipeline, library browser, pagination fix, project manager, and E2E test infrastructure. The v004 retrospective (completed 2026-02-09) confirms a strong testing foundation with 571 tests at 92.86% coverage, validating the "infrastructure first" sequencing strategy that should be applied to v005.

## Backlog Overview

- **Total items:** 10
- **Priority distribution:** 6x P1, 4x P2
- **All items size M**
- **All items status: open**

| Priority | Count | Items |
|----------|-------|-------|
| P1 | 6 | BL-028, BL-029, BL-030, BL-032, BL-033, BL-035 |
| P2 | 4 | BL-003, BL-031, BL-034, BL-036 |

## Previous Version

- **Version:** v004 — Testing Infrastructure & Quality Verification
- **Completed:** 2026-02-09
- **Retrospective:** `comms/outbox/versions/execution/v004/retrospective.md`
- **Stats:** 5 themes, 15 features, 70/70 acceptance criteria, 178 new tests

## Key Learnings Applicable to v005

1. **Build infrastructure first** (LRN-019): Sequence frontend scaffolding and WebSocket before GUI panels
2. **Validate AC against codebase** (LRN-016): Ensure acceptance criteria reference APIs that actually exist
3. **Constructor DI pattern** (LRN-005): Continue `create_app()` DI for WebSocket and new endpoint testing
4. **PyO3 FFI overhead** (LRN-012): Keep Rust for string-heavy/safety operations; Python for simple logic
5. **Python/Rust boundaries** (LRN-011): Rust handles input safety, Python handles business policy

## Tech Debt to Address

- **BL-034 (pagination total count):** Identified in v003 retrospective, directly blocks library browser virtual scrolling
- **Rust coverage at 75%** (target 90%): Deferred from v004, not in v005 scope but should be tracked
- **Scan endpoint lacks blackbox tests:** Requires real video files; may affect E2E test design

## Missing Items

None. All 10 backlog items listed in PLAN.md were found and fully detailed.
