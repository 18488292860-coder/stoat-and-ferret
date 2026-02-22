# Draft Checklist — v009

- [x] manifest.json is valid JSON with all required fields
- [x] Every theme in manifest has a corresponding folder under drafts/ (observability-pipeline/, gui-runtime-fixes/)
- [x] Every feature in manifest has a corresponding folder with both requirements.md and implementation-plan.md
- [x] VERSION_DESIGN.md and THEME_INDEX.md exist in drafts/
- [x] THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+`
- [x] No placeholder text in any draft (`_Theme goal_`, `_Feature description_`, `[FILL IN]`, `TODO`)
- [x] All backlog IDs from manifest appear in at least one requirements.md (BL-057, BL-059, BL-060, BL-063, BL-064, BL-065)
- [x] No theme or feature slug starts with a digit prefix (`^\d+-`)
- [x] Backlog IDs in each requirements.md cross-referenced against Task 002 backlog analysis (no mismatches)
- [x] All "Files to Modify" paths verified via `request_clarification` structure query (no unverified paths)
- [x] No feature requirements.md or implementation-plan.md instructs MCP tool calls (features execute with Read/Write/Edit/Bash only)

## Path Corrections Applied

Two file paths were corrected during verification:

1. **file-logging implementation-plan.md:** `src/stoat_ferret/settings.py` → `src/stoat_ferret/api/settings.py`
2. **websocket-broadcasts implementation-plan.md:** `src/stoat_ferret/services/scan_service.py` → `src/stoat_ferret/api/services/scan.py`

## Backlog ID Cross-Reference

| Feature | BL ID | Verified Against |
|---------|-------|-----------------|
| ffmpeg-observability | BL-059 | Task 002 backlog-details.md: "Wire ObservableFFmpegExecutor into dependency injection" |
| audit-logging | BL-060 | Task 002 backlog-details.md: "Wire AuditLogger into repository dependency injection" |
| file-logging | BL-057 | Task 002 backlog-details.md: "Add file-based logging with rotation to logs/ directory" |
| spa-routing | BL-063 | Task 002 backlog-details.md: "Add SPA routing fallback for GUI sub-paths" |
| pagination-fix | BL-064 | Task 002 backlog-details.md: "Fix projects endpoint pagination total count" |
| websocket-broadcasts | BL-065 | Task 002 backlog-details.md: "Wire WebSocket broadcast calls into API operations" |

## File Action Corrections

- `tests/test_audit_logging.py`: Changed from "Create" to "Modify" (file already exists)
