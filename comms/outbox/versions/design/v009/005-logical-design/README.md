# 005 Logical Design — v009

v009 proposes 2 themes with 6 features total, all wiring gaps from earlier versions. Theme 1 (01-observability-pipeline, 3 features) wires existing but unused observability components into DI and startup. Theme 2 (02-gui-runtime-fixes, 3 features) fixes runtime gaps in SPA routing, pagination, and WebSocket broadcasts. All 6 backlog items (BL-057, BL-059, BL-060, BL-063, BL-064, BL-065) are mapped — no deferrals.

## Theme Overview

### Theme 1: 01-observability-pipeline
**Goal:** Wire ObservableFFmpegExecutor, AuditLogger, and RotatingFileHandler into the application's DI chain and startup sequence.

| Feature | Backlog | Goal |
|---------|---------|------|
| 001-ffmpeg-observability | BL-059 (P1) | Wire ObservableFFmpegExecutor into DI for metrics and structured logs |
| 002-audit-logging | BL-060 (P2) | Wire AuditLogger into repository DI for audit trail |
| 003-file-logging | BL-057 (P2) | Add RotatingFileHandler for persistent log output |

### Theme 2: 02-gui-runtime-fixes
**Goal:** Fix SPA routing fallback, pagination total count, and WebSocket broadcast calls in the API/GUI layer.

| Feature | Backlog | Goal |
|---------|---------|------|
| 001-spa-routing | BL-063 (P1) | Add catch-all SPA fallback route for /gui/* sub-paths |
| 002-pagination-fix | BL-064 (P2) | Add count() to AsyncProjectRepository and fix total |
| 003-websocket-broadcasts | BL-065 (P2) | Wire broadcast() calls into scan and project operations |

## Key Decisions

1. **Theme grouping by modification point** (LRN-042): Theme 1 modifies DI/startup code, Theme 2 modifies API/GUI layer. Avoids cross-theme conflicts in `app.py`.

2. **SPA fallback via catch-all route**: Research confirmed `StaticFiles(html=True)` does NOT provide SPA fallback. A FastAPI catch-all route is needed, conditional on `gui/dist/` existing (LRN-020).

3. **Constructor DI pattern** (LRN-005): BL-059 and BL-060 follow the established `create_app()` kwargs pattern, adding `ffmpeg_executor` and `audit_logger` parameters.

4. **No new MCP handlers**: All changes are to application-level FastAPI endpoints and internal wiring. Handler concurrency evaluation is not applicable.

## Dependencies

- **Theme 1 before Theme 2**: Sequential to avoid concurrent `app.py` modifications. No functional dependency.
- **Within themes**: Features are sequenced to avoid merge conflicts in shared files (`app.py`, `create_app()`), but have no functional inter-dependencies.
- **External**: BL-057 depends on BL-056 (structured logging, completed in v008). All other items are independent.

## Risks and Unknowns

| Risk | Severity | Summary |
|------|----------|---------|
| WebSocket broadcast cross-cutting | Medium | BL-065 touches multiple routers — risk of partial or inconsistent implementation |
| SPA route ordering | Medium | Catch-all route must not intercept static asset requests |
| AuditLogger sync connection | Medium | AuditLogger takes synchronous sqlite3.Connection but repos use aiosqlite |
| Lifespan complexity growth | Low | 3 more concerns added to lifespan — monitor size |
| Log backup count setting | Low | May need new Settings field — must wire consumer immediately |
| Test double injection | Low | Observable executor must not wrap test doubles |
| Frontend pagination edge cases | Low | Changing total value — should be safe but unverified for projects |

See `risks-and-unknowns.md` for full details. These feed into Task 006 (Critical Thinking).

## Output Files

- `logical-design.md` — Complete theme/feature breakdown, execution order, research sources, handler concurrency
- `test-strategy.md` — Test requirements per feature (unit, integration, E2E, parity, contract)
- `risks-and-unknowns.md` — All identified risks with severity, investigation needs, and assumptions
