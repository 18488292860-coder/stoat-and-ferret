# Task 003: Impact Assessment — v008

16 impacts identified across 4 backlog items: 15 small (sub-task scope), 1 substantial (feature scope), 0 cross-version. All v008 changes are self-contained wiring fixes with well-bounded impacts.

## Generic Impacts

- **Documentation (4 impacts):** Setup guide (`02_development-setup.md`) and configuration docs (`04_configuration.md`) reference behavior that will change. C4 architecture docs describe lifespan without logging or schema creation steps, and their Mermaid diagrams omit these dependencies. All are small updates that can be handled as sub-tasks within the causing feature.

- **Caller-impact (9 impacts):** Type conversion needed between `settings.log_level` (string) and `configure_logging(level: int)`. Async/sync bridge needed for `create_tables()` in the async lifespan. Three duplicate `create_tables_async()` helpers in test files will grow to four copies — the only substantial impact, warranting extraction into a shared module.

- **Test compatibility (2 impacts):** Newly-active logging output may affect tests expecting silent stdout. Schema creation in lifespan must respect the existing `_deps_injected` test bypass.

## Project-Specific Impacts

N/A — no `docs/auto-dev/IMPACT_ASSESSMENT.md` exists for this project.

## Work Items Generated

| Classification | Count | Details |
|---------------|-------|---------|
| Small | 15 | Doc updates (4), caller wiring sub-tasks (8), test verification (2), E2E timeout (1) |
| Substantial | 1 | Extract shared `create_tables_async()` to eliminate 4-way DDL duplication |
| Cross-version | 0 | — |

## Recommendations

1. **BL-058 should include the `create_tables_async()` extraction** as a distinct sub-task or feature. Three test files already duplicate this function, and BL-058 adds a fourth consumer. Per LRN-035, the third duplication is the trigger for extraction. This is the only substantial impact and should be scoped explicitly in logical design.

2. **Documentation updates are small and co-located** — each can be a final sub-task within its causing feature. No separate documentation feature is needed.

3. **BL-056 and BL-062 share the `__main__.py` change** — both modify `uvicorn.run()` kwargs (BL-056 for `log_level`, BL-062 for `debug`). Sequence BL-056 before BL-062 or combine into a single feature to avoid merge conflicts.

4. **All caller-impact items are implementation details**, not design surprises. The type conversion, async bridge, and settings access patterns are standard and can be handled within normal feature implementation.

5. **Test compatibility checks are low-risk** — the `_deps_injected` bypass is well-established and existing test patterns are compatible with the planned changes. Include verification as acceptance criteria rather than separate work items.
