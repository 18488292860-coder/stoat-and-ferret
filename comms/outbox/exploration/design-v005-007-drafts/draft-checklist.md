# Draft Checklist - v005 Design Documents

## Structural Completeness

- [x] manifest.json is valid JSON with all required fields
- [x] Every theme in manifest has a corresponding folder under drafts/
- [x] Every feature in manifest has both requirements.md and implementation-plan.md
- [x] VERSION_DESIGN.md and THEME_INDEX.md exist in drafts/
- [x] THEME_INDEX feature lines match format `- \d{3}-[\w-]+: .+`
- [x] No placeholder text in any draft
- [x] All backlog IDs from manifest appear in at least one requirements.md
- [x] No theme or feature slug starts with a digit prefix
- [x] Backlog IDs in each requirements.md cross-referenced against Task 002 backlog analysis

## Backlog ID Cross-Reference

| Backlog ID | In manifest.json | In requirements.md | Verified against backlog-details.md |
|------------|-------------------|--------------------|------------------------------------|
| BL-003 | Yes | frontend-scaffolding, settings-and-docs | Yes - "EXP-003: FastAPI static file serving for GUI" |
| BL-028 | Yes | frontend-scaffolding | Yes - "Frontend framework selection and Vite project setup" |
| BL-029 | Yes | websocket-endpoint, settings-and-docs | Yes - "WebSocket endpoint for real-time events" |
| BL-030 | Yes | application-shell | Yes - "Application shell and navigation components" |
| BL-031 | Yes | dashboard-panel | Yes - "Dashboard panel with health cards and activity log" |
| BL-032 | Yes | thumbnail-pipeline | Yes - "Thumbnail generation pipeline for video library" |
| BL-033 | Yes | library-browser | Yes - "Library browser with video grid, search, and scan UI" |
| BL-034 | Yes | pagination-total-count | Yes - "Fix pagination total count for list endpoints" |
| BL-035 | Yes | project-manager | Yes - "Project manager with list, creation, and details views" |
| BL-036 | Yes | playwright-setup, e2e-test-suite | Yes - "Playwright E2E test infrastructure for GUI" |

## Slug Verification

### Theme slugs (no number prefixes):
- `frontend-foundation` (not `01-frontend-foundation`)
- `backend-services` (not `02-backend-services`)
- `gui-components` (not `03-gui-components`)
- `e2e-testing` (not `04-e2e-testing`)

### Feature slugs (no number prefixes):
- `frontend-scaffolding` (not `001-frontend-scaffolding`)
- `websocket-endpoint` (not `002-websocket-endpoint`)
- `settings-and-docs` (not `003-settings-and-docs`)
- `thumbnail-pipeline` (not `001-thumbnail-pipeline`)
- `pagination-total-count` (not `002-pagination-total-count`)
- `application-shell` (not `001-application-shell`)
- `dashboard-panel` (not `002-dashboard-panel`)
- `library-browser` (not `003-library-browser`)
- `project-manager` (not `004-project-manager`)
- `playwright-setup` (not `001-playwright-setup`)
- `e2e-test-suite` (not `002-e2e-test-suite`)

## THEME_INDEX Format Verification

Each feature line verified against parser regex `- (\d+)-([\w-]+):`:

```
- 001-frontend-scaffolding: ...  -> matches (\d+)-([\w-]+): YES
- 002-websocket-endpoint: ...    -> matches (\d+)-([\w-]+): YES
- 003-settings-and-docs: ...     -> matches (\d+)-([\w-]+): YES
- 001-thumbnail-pipeline: ...    -> matches (\d+)-([\w-]+): YES
- 002-pagination-total-count: .. -> matches (\d+)-([\w-]+): YES
- 001-application-shell: ...     -> matches (\d+)-([\w-]+): YES
- 002-dashboard-panel: ...       -> matches (\d+)-([\w-]+): YES
- 003-library-browser: ...       -> matches (\d+)-([\w-]+): YES
- 004-project-manager: ...       -> matches (\d+)-([\w-]+): YES
- 001-playwright-setup: ...      -> matches (\d+)-([\w-]+): YES
- 002-e2e-test-suite: ...        -> matches (\d+)-([\w-]+): YES
```

No forbidden formats used (no numbered lists, no bold identifiers, no metadata before colon, no multi-line entries).

## Document Content Verification

- [x] All requirements.md include: Goal, Background, Backlog Item, Functional Requirements (FR-xxx), Non-Functional Requirements (NFR-xxx), Out of Scope, Test Requirements, Reference
- [x] All implementation-plan.md include: Overview, Files to Create/Modify, Implementation Stages with verification commands, Test Infrastructure Updates, Quality Gates, Risks, Commit Message
- [x] All THEME_DESIGN.md include: Goal, Backlog Items, Features table, Dependencies, Technical Approach, Risks
- [x] VERSION_DESIGN.md includes: Overview, Goals, Scope, Themes, Execution Order, Key Technical Decisions, Dependencies, Risks, Constraints, References
- [x] Test requirements align with Task 005/006 test strategy
