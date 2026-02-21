# Feature: orphaned-settings

## Goal

Wire settings.debug to FastAPI and settings.ws_heartbeat_interval to ws.py so all defined settings are consumed.

## Background

**Backlog Item:** BL-062 (P2)

Two settings fields are defined with full validation and env var support but never consumed: `debug` is never passed to FastAPI or uvicorn, and `ws_heartbeat_interval` is ignored in favour of a hardcoded `DEFAULT_HEARTBEAT_INTERVAL = 30` in ws.py. These settings were created in v003/02-api-foundation as "define the field" without "wire the field to its consumer."

Task 006 confirmed: `get_settings()` is `@lru_cache`-decorated and safe for runtime access. The heartbeat interval is read at connection time (inside `websocket_endpoint()` handler), not at module import time. DEFAULT_HEARTBEAT_INTERVAL (30) matches settings default (30).

## Functional Requirements

**FR-001:** settings.debug controls FastAPI debug mode
- Acceptance: settings.debug is passed to FastAPI(debug=...) constructor

**FR-002:** settings.ws_heartbeat_interval controls WebSocket heartbeat
- Acceptance: settings.ws_heartbeat_interval is read by ws.py instead of using hardcoded DEFAULT_HEARTBEAT_INTERVAL

**FR-003:** No settings fields remain unconsumed
- Acceptance: No settings fields remain defined but unconsumed (excluding log_level which is covered by BL-056)

## Non-Functional Requirements

**NFR-001:** Default behavior unchanged
- Metric: With default settings, application behavior is identical to pre-v008 (debug=False, heartbeat=30s)

## Handler Pattern

Not applicable for v008 — no new handlers introduced.

## Out of Scope

- Adding new settings fields
- Settings validation changes
- Environment variable documentation updates (co-located sub-task, not a separate feature)

## Test Requirements

**Unit tests:**
- Test FastAPI app created with debug=True when settings.debug=True
- Test FastAPI app created with debug=False when settings.debug=False (default)
- Test ws.py reads ws_heartbeat_interval from settings instead of constant
- Test that DEFAULT_HEARTBEAT_INTERVAL constant is removed or no longer referenced

**System tests:**
- Test STOAT_DEBUG=true enables FastAPI debug mode (verify app.debug attribute)
- Test STOAT_WS_HEARTBEAT_INTERVAL=15 changes heartbeat loop interval

**Existing test modifications:**
- None expected (settings wiring is additive; defaults match existing behavior)

## Reference

See `comms/outbox/versions/design/v008/004-research/` for supporting evidence.
