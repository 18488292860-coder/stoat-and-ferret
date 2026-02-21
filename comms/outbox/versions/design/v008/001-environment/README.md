# Task 001: Environment Verification — v008

Environment is **ready** for v008 design. All health checks pass, the project is properly configured with 24 completed themes and no active execution. The working tree is on `main`, synced with remote, and the only modified file is the auto-dev exploration state tracker (expected). C4 architecture documentation is current through v007.

## Environment Status

- **MCP Server:** Healthy (v6.0.0, uptime 11101s)
- **Services:** config=ok, state=ok, execution=ok
- **External Dependencies:** git available, gh CLI available and authenticated
- **Source Sync:** Checksums match — running code matches source
- **Tool Authorization:** Enabled, 4 active keys, no orphaned keys
- **No critical errors or warnings**

## Project Status

- **Project:** stoat-and-ferret (AI-driven video editor, hybrid Python/Rust)
- **Active Themes:** None (idle, ready for new version)
- **Completed Themes:** 24 across 7 versions (v001-v007)
- **Destructive Test Target:** false
- **Execution Backend:** legacy mode

## Git Status

- **Branch:** `main` tracking `origin/main`
- **Ahead/Behind:** 0/0 (fully synced)
- **Working Tree:** 1 modified file (auto-dev exploration state tracker — expected)
- **Staged/Untracked:** None
- **Remote:** https://github.com/gwickman/stoat-and-ferret.git

## Architecture Context

- **C4 Documentation:** Present and current through v007 (last updated 2026-02-19)
- **Generation Mode:** Delta updates applied incrementally (full baseline at v005)
- **Coverage:** 54 documents total — context, container, component (8), code-level (42)
- **API Spec:** OpenAPI 3.1 at v0.7.0
- **Key Components:** Rust Core Engine, Python Bindings, Effects Engine, API Gateway, Application Services, Data Access, Web GUI, Test Infrastructure

## Version Scope

**v008 — Startup Integrity & CI Stability**

Goal: Fix P0 blockers and critical startup wiring gaps. After v008, a fresh install starts cleanly with working logging, database, and settings.

**Backlog Items:** BL-055, BL-056, BL-058, BL-062 (4 items)

**Themes:**
1. **application-startup-wiring** (3 features)
   - 001-database-startup: Wire `create_tables()` into lifespan [BL-058, P0]
   - 002-logging-startup: Call `configure_logging()` at startup [BL-056, P1]
   - 003-orphaned-settings: Wire `settings.debug` and `settings.ws_heartbeat_interval` [BL-062, P2]
2. **ci-stability** (1 feature)
   - 001-flaky-e2e-fix: Fix Playwright timeout flake [BL-055, P0]

**Dependencies:** None. All items are independent startup/CI fixes.
**Risk:** BL-055 (flaky E2E) may require Playwright timing investigation.
**Deferred from prior versions:** None directly relevant to v008 scope.
