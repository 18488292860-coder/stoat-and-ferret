# Draft Checklist: v008

- [x] manifest.json is valid JSON with all required fields
- [x] Every theme in manifest has a corresponding folder under drafts/
- [x] Every feature in manifest has a corresponding folder with both requirements.md and implementation-plan.md
- [x] VERSION_DESIGN.md and THEME_INDEX.md exist in drafts/
- [x] THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+`
- [x] No placeholder text in any draft (`_Theme goal_`, `_Feature description_`, `[FILL IN]`, `TODO`)
- [x] All backlog IDs from manifest appear in at least one requirements.md
- [x] No theme or feature slug starts with a digit prefix (`^\d+-`)
- [x] Backlog IDs in each requirements.md cross-referenced against Task 002 backlog analysis (no mismatches)
- [x] All "Files to Modify" paths verified via `request_clarification` structure query (no unverified paths)

## Verification Details

### Backlog ID Cross-Reference
| requirements.md | Backlog ID Used | Task 002 Confirms | Match |
|-----------------|-----------------|-------------------|-------|
| database-startup | BL-058 | BL-058: Wire create_tables() into lifespan startup | Yes |
| logging-startup | BL-056 | BL-056: Wire up structured logging at application startup | Yes |
| orphaned-settings | BL-062 | BL-062: Wire orphaned settings (debug, ws_heartbeat_interval) | Yes |
| flaky-e2e-fix | BL-055 | BL-055: Fix flaky E2E test in project-creation.spec.ts | Yes |

### File Path Verification
All "Files to Modify" paths confirmed via structure query and Glob:
- `src/stoat_ferret/db/schema.py` — exists
- `src/stoat_ferret/api/app.py` — exists
- `src/stoat_ferret/logging.py` — exists
- `src/stoat_ferret/api/__main__.py` — exists
- `src/stoat_ferret/api/routers/ws.py` — exists
- `tests/test_async_repository_contract.py` — exists
- `tests/test_project_repository_contract.py` — exists
- `tests/test_clip_repository_contract.py` — exists
- `tests/test_logging.py` — exists
- `gui/e2e/project-creation.spec.ts` — exists

### Slug Verification
Theme slugs: `application-startup-wiring`, `ci-stability` — no digit prefixes
Feature slugs: `database-startup`, `logging-startup`, `orphaned-settings`, `flaky-e2e-fix` — no digit prefixes
