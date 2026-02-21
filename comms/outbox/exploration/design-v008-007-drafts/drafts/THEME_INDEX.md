# Theme Index: v008

## 01-application-startup-wiring

Wire existing but disconnected infrastructure — database schema creation, structured logging, and orphaned settings — into the FastAPI lifespan startup sequence.

**Features:**

- 001-database-startup: Wire create_tables() into lifespan so database schema is created automatically on startup
- 002-logging-startup: Call configure_logging() at startup and wire settings.log_level to control log verbosity
- 003-orphaned-settings: Wire settings.debug to FastAPI and settings.ws_heartbeat_interval to ws.py

## 02-ci-stability

Eliminate the flaky E2E test that intermittently blocks CI merges.

**Features:**

- 001-flaky-e2e-fix: Fix toBeHidden() timeout in project-creation.spec.ts so E2E passes reliably
