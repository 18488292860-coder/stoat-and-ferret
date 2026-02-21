# Backlog Details — v008

## BL-055: Fix flaky E2E test in project-creation.spec.ts (toBeHidden timeout)

- **Priority:** P0
- **Size:** L
- **Tags:** bug, e2e, ci, flaky-test
- **Status:** open

**Description:**

> The E2E test at gui/e2e/project-creation.spec.ts:31 intermittently fails with a toBeHidden assertion timeout on the project creation modal. The test fails on main (GitHub Actions run 22188785818) independent of any feature branch changes. During v007 execution, this caused the dynamic-parameter-forms feature to receive a 'partial' completion status despite 12/12 acceptance criteria passing and all other quality gates green. The execution pipeline halted, requiring manual restart. Any future feature touching the E2E CI job is at risk of the same false-positive halt.

**Acceptance Criteria:**

1. project-creation.spec.ts:31 toBeHidden assertion passes reliably across 10 consecutive CI runs
2. No E2E test requires retry loops to pass in CI
3. Flaky test fix does not alter the tested project creation functionality

**Notes:**

> Discovered during v007 execution. PR #88 (dynamic-parameter-forms) was retried 3 times per AGENTS.md limit without resolution. Likely cause: timing-dependent modal animation or state cleanup between tests.

**Complexity Assessment:** Medium-high. Requires diagnosing a Playwright timing issue (modal animation/state cleanup race condition). The root cause is not definitively known — investigation is needed. Fix must preserve existing functionality while eliminating the flake. Proving reliability (10 consecutive CI runs) adds validation overhead.

---

## BL-056: Wire up structured logging at application startup

- **Priority:** P1
- **Size:** XL
- **Tags:** observability, logging, wiring-gap
- **Status:** open

**Description:**

> **Current state:** `configure_logging()` exists in `src/stoat_ferret/logging.py` and 10 modules emit structlog calls, but `configure_logging()` is never called at startup. `settings.log_level` (via `STOAT_LOG_LEVEL` env var) is defined but never consumed. As a result, all `logger.info()` calls are silently dropped (Python's `lastResort` handler only shows WARNING+), and the only functioning log output is uvicorn's hardcoded `log_level="info"`.
>
> **Gap:** The logging infrastructure was built in v002/v003 but never wired together. The function, the setting, and the log call sites all exist independently with no connection between them.
>
> **Impact:** No application-level logging is visible. Correlation IDs, job lifecycle events, effect registration, websocket events, and all structured log data are lost. Debugging production issues requires code changes to enable any logging.

**Acceptance Criteria:**

1. Application calls configure_logging() during startup lifespan before any request handling
2. settings.log_level value is passed to configure_logging() and controls the root logger level
3. STOAT_LOG_LEVEL=DEBUG produces visible debug output on stdout
4. All existing logger.info() calls across the 10 modules produce visible structured output at INFO level
5. uvicorn log_level uses settings.log_level instead of hardcoded 'info'
6. Existing tests continue to pass with logging active

**Notes:**

> Scope: wires up existing stdout-only logging infrastructure. File-based logging is a separate follow-on item.

**Use Case:**

> During development and incident debugging, any developer or the auto-dev execution pipeline needs visible structured log output to diagnose failures without modifying code.

**Complexity Assessment:** Medium. The function and all call sites exist — this is pure wiring. However, 6 ACs touching the lifespan, settings, uvicorn config, and 10 existing modules suggest broad surface area. The main risk is test interference from newly-active logging output.

---

## BL-058: Wire database schema creation into application startup

- **Priority:** P0
- **Size:** L
- **Tags:** wiring-gap, database, startup
- **Status:** open

**Description:**

> **Current state:** `create_tables()` exists and Alembic migrations are configured, but neither is called during application startup. A fresh database gets no schema — the application starts but any database operation will fail.
>
> **Gap:** Database initialization was built in v002/03-database-foundation but never wired into the lifespan startup sequence. Requires manual CLI intervention to create schema.
>
> **Impact:** The application cannot function against a new database without manual steps. This contradicts the project's principle that all processes must be fully automated.

**Acceptance Criteria:**

1. Database tables are created automatically during application startup lifespan if they don't exist
2. Alembic migrations run at startup or a clear automated mechanism ensures schema is current
3. A fresh database with no prior state becomes fully functional after a single application start
4. Existing tests continue to pass

**Complexity Assessment:** Low-medium. `create_tables()` already exists and the lifespan startup hook is an established pattern. The main design decision is whether to use `create_tables()` directly or integrate Alembic migrations at startup. AC #2 allows either approach.

---

## BL-062: Wire orphaned settings (debug, ws_heartbeat_interval) to their consumers

- **Priority:** P2
- **Size:** L
- **Tags:** wiring-gap, settings, configuration
- **Status:** open

**Description:**

> **Current state:** Two settings fields are defined with full validation and env var support but never consumed: `debug` is never passed to FastAPI or uvicorn, and `ws_heartbeat_interval` is ignored in favour of a hardcoded `DEFAULT_HEARTBEAT_INTERVAL = 30` in ws.py.
>
> **Gap:** Settings were created in v003/02-api-foundation as "define the field" without "wire the field to its consumer."
>
> **Impact:** Operators cannot control debug mode or WebSocket heartbeat timing via configuration. The env vars exist but do nothing.

**Acceptance Criteria:**

1. settings.debug is passed to FastAPI(debug=...) and/or uvicorn.run()
2. settings.ws_heartbeat_interval is read by ws.py instead of using hardcoded DEFAULT_HEARTBEAT_INTERVAL
3. No settings fields remain defined but unconsumed (excluding log_level which is covered by BL-056)

**Complexity Assessment:** Low. Two straightforward wiring changes: pass `settings.debug` to FastAPI constructor, replace the hardcoded constant in ws.py with `settings.ws_heartbeat_interval`. AC #3 requires auditing all settings fields, which adds minor scope.
