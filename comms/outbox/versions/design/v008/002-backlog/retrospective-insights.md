# Retrospective Insights — v007 to v008

Synthesized from `comms/outbox/versions/execution/v007/retrospective.md` and `docs/versions/v007/VERSION_SUMMARY.md`.

## What Worked Well (Continue in v008)

1. **Current-state/gap/impact description format.** All v008 backlog items follow this structure, which enabled first-iteration success across v006 and v007. Continue using this format for design specifications.

2. **Clean acceptance criteria with verifiable actions.** v007 achieved 131/131 AC pass rate. v008 items follow the same verb + observable outcome pattern.

3. **Independent feature sequencing.** v007's four Zustand stores composed cleanly because features were designed as standalone units. v008's four items are also fully independent — no inter-feature dependencies.

4. **Documentation proximity to implementation.** Architecture docs updated immediately after implementation (v007 themes 02, 04) stayed accurate. v008 scope is small enough that doc updates may not be needed, but the pattern should be applied if any API changes occur.

## What Didn't Work (Avoid in v008)

1. **Flaky E2E test blocking PR merges.** The `project-creation.spec.ts:31` flake blocked two v007 themes (03 and 04), caused a "partial" completion status on dynamic-parameter-forms despite all ACs passing, and required manual pipeline restarts. **v008 directly addresses this with BL-055.**

2. **Missing wiring audit before feature development.** The startup wiring gaps (BL-056, BL-058, BL-062) existed since v002/v003 but were not discovered until a post-v007 wiring audit. Earlier audits would have caught these sooner. **v008 directly addresses the three most critical gaps.**

## Tech Debt Addressed vs Deferred

### Addressed in v008

| Debt Item | Source | v008 Item |
|-----------|--------|-----------|
| Flaky E2E test | v007 retro, high priority | BL-055 |
| Logging not wired | Post-v007 audit | BL-056 |
| Database startup not wired | Post-v007 audit | BL-058 |
| Orphaned settings | Post-v007 audit | BL-062 |

### Deferred (Not in v008 Scope)

| Debt Item | Priority | Target |
|-----------|----------|--------|
| SPA fallback routing (BL-063) | Medium | v009 |
| Large definitions.py module (~576 lines) | Medium | Future (when adding more effects) |
| EffectsPage orchestrator complexity | Medium | Future |
| Manual enum stub maintenance | Medium | Future (after 3rd enum type) |
| Stub file duplication | Low | Future |
| Transition storage as JSON column | Low | Future |
| Preview thumbnail (M2.9 sub-5) | Low | Future |

## Architectural Decisions Informing v008

1. **Lifespan startup pattern is established.** The FastAPI lifespan context manager is the canonical location for startup tasks (existing pattern from v003). Both BL-056 and BL-058 should wire into this same lifespan function.

2. **Settings pattern is established.** `AppSettings` with Pydantic validation and env var support is the canonical settings source (v003). BL-062 wires existing settings to consumers — no new patterns needed.

3. **DI via `create_app()` kwargs on `app.state`.** This is the established dependency injection pattern. Startup wiring should use settings from `app.state` rather than importing settings globally.

4. **Test mode bypasses lifespan** via `app.state._deps_injected = True`. BL-056 and BL-058 implementations must ensure test mode compatibility — startup tasks should be skippable when deps are pre-injected.

## Process Recommendations for v008

1. **Sequence BL-058 (database) before BL-056 (logging)** within Theme 1, since logging initialization may emit during database startup.
2. **Validate file paths in ACs against codebase** before finalizing design (per LRN-016). Confirm `configure_logging()`, `create_tables()`, `settings.debug`, `ws.py` `DEFAULT_HEARTBEAT_INTERVAL` all exist as described.
3. **Fix BL-055 early** (Theme 2) to unblock any CI-dependent work, per LRN-033.
