# Smoke Test Key Files

> **Read this file first, then read only the files relevant to your task. Do not read all files listed here unless necessary.**

## What is the Smoke Test Suite?

The smoke test suite is an API-level integration test suite that runs in-process using `httpx.ASGITransport` (no real HTTP server). It contains 13 tests covering 12 use cases and exercises the full application stack including the real Rust core (`stoat_ferret_core`). Tests verify API endpoint contracts, request/response schemas, and end-to-end behaviour from HTTP request through to Rust-level processing.

## Key Files

| File | What it tells you |
|------|-------------------|
| `tests/smoke/conftest.py` | Shared fixtures, test video setup, ASGITransport client |
| `tests/smoke/test_scan_workflow.py` | UC-01 and UC-12: video scanning and job cancellation |
| `tests/smoke/test_library.py` | UC-02: library search |
| `tests/smoke/test_project_workflow.py` | UC-03 and UC-09: project CRUD and deletion |
| `tests/smoke/test_clip_workflow.py` | UC-04 and UC-10: clip add and modify |
| `tests/smoke/test_effects.py` | UC-05, UC-06, UC-11: effects catalog, update/delete, speed and stacking |
| `tests/smoke/test_transitions.py` | UC-07: fade transition |
| `tests/smoke/test_health.py` | UC-08: health live and ready probes |
| `docs/setup/smoke-test-harness-guide/03-test-cases.md` | Canonical use case definitions |
| `docs/setup/smoke-test-harness-guide/02-infrastructure.md` | Harness architecture decisions |
