# Impact Summary — v008

## Small Impacts (sub-task scope)

### Documentation Updates (owned by the feature that causes the change)

1. **Update development setup guide** — `docs/setup/02_development-setup.md` Section 5 currently describes manual `alembic upgrade head` as required; after BL-058, schema is auto-created at startup. Update to note this and keep Alembic commands for migration management reference. *(BL-058)*

2. **Update configuration docs** — `docs/setup/04_configuration.md` lines 26-27 describe `STOAT_DEBUG` and `STOAT_LOG_LEVEL` without noting their runtime effect. After BL-056/BL-062, update descriptions to explain what these settings control. *(BL-056, BL-062)*

3. **Update C4 lifespan descriptions** — `c4-code-stoat-ferret-api.md` line 18 and `c4-code-python-api.md` line 19 describe lifespan without logging or schema creation. Update both after BL-056/BL-058. *(BL-056, BL-058)*

4. **Update C4 Mermaid diagrams** — Two Mermaid diagrams in `c4-code-stoat-ferret-api.md` and `c4-code-python-api.md` show lifespan dependencies without logging or schema nodes. Add edges after BL-056/BL-058. *(BL-056, BL-058)*

### Caller Wiring (implementation sub-tasks within each feature)

5. **Wire log_level to uvicorn** — `__main__.py` line 27 hardcodes `log_level="info"`. Change to `settings.log_level.lower()`. *(BL-056)*

6. **Type conversion for configure_logging** — `configure_logging()` accepts `level: int` but settings provides a string literal. Add `getattr(logging, settings.log_level)` conversion. *(BL-056)*

7. **Async/sync bridge for create_tables** — `create_tables()` is synchronous but lifespan is async. Use `run_in_executor()` or rewrite as async using the `await conn.execute()` pattern from test helpers. *(BL-058)*

8. **Wire ws_heartbeat_interval** — Replace `DEFAULT_HEARTBEAT_INTERVAL` constant in `ws.py` with `get_settings().ws_heartbeat_interval`. *(BL-062)*

9. **Wire debug to FastAPI** — Add `debug=settings.debug` to `FastAPI()` constructor in `create_app()`. *(BL-062)*

10. **Settings audit verification** — After BL-056 and BL-062, verify all 9 settings fields are consumed. Expected: all consumed. *(BL-062)*

11. **E2E timeout alignment** — `project-creation.spec.ts` uses default 5s timeout while other E2E tests use 10-15s. Fix the flaky assertion. *(BL-055)*

### Test Compatibility (verification sub-tasks)

12. **Logging output in tests** — BL-056 activates previously-silent logging. Verify no tests assert on stdout expecting silence. *(BL-056)*

13. **Test mode schema bypass** — BL-058 adds schema creation to lifespan. Verify `_deps_injected=True` bypass prevents unwanted schema creation in tests. *(BL-058)*

## Substantial Impacts (feature scope)

14. **Extract shared async schema creation helper** — Three async test files (`test_async_repository_contract.py`, `test_project_repository_contract.py`, `test_clip_repository_contract.py`) each duplicate a `create_tables_async()` function with manually-listed DDL statements. BL-058 will add a fourth copy (in `app.py` lifespan). Per LRN-035 ("automate when third instance appears"), extract a shared `create_tables_async()` into `src/stoat_ferret/db/schema.py` alongside the existing sync `create_tables()`, and import it in all four locations. This eliminates the maintenance risk of DDL statements drifting out of sync across copies. *(BL-058)*

## Cross-Version Impacts (backlog scope)

None identified. All v008 changes are self-contained wiring fixes with no architectural spillover.
