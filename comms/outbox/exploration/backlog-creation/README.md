# Exploration: Backlog Creation for v004–v007

## Summary

Created **33 new backlog items** (BL-020 through BL-052) across v004–v007 based on the gap analysis at `comms/outbox/exploration/backlog-gap-analysis/`. Updated **5 existing items** with version tags. Updated `docs/auto-dev/plan.md` with backlog coverage table, cross-version dependencies, scoping decisions for v006/v007, and exploration prerequisites.

## Items Created by Version

| Version | Milestones | Items Created | IDs | Priority Breakdown |
|---------|-----------|---------------|-----|-------------------|
| v004 | M1.8–1.9 | 8 | BL-020–027 | 4×P1, 3×P2, 1×P3 |
| v005 | M1.10–1.12 | 9 | BL-028–036 | 5×P1, 3×P2, 1×P2 |
| v006 | M2.1–2.3 | 7 | BL-037–043 | 5×P1, 2×P2 |
| v007 | M2.4–2.6, 2.8–2.9 | 9 | BL-044–052 | 7×P1, 1×P1, 1×P2 |
| **Total** | | **33** | | |

## Existing Items Updated

5 open backlog items tagged with `v004`:
- BL-009: Add property test guidance to feature design template
- BL-010: Configure Rust code coverage with llvm-cov
- BL-012: Fix coverage reporting gaps for ImportError fallback
- BL-014: Add Docker-based local testing option
- BL-016: Unify InMemory vs FTS5 search behavior

## Plan.md Updates

- Added "Backlog Coverage by Version" table to Backlog Integration section
- Added cross-version dependency notes
- Added exploration prerequisites list (4 items needing EXP before implementation)
- Added v006 and v007 scoping boundary sections

## Issues Encountered

None. All 33 items created successfully via `add_backlog_item` MCP tool. All 5 updates applied via `update_backlog_item`.

## Gap Analysis Coverage

The gap analysis suggested 33 items total (8+9+7+9). All 33 were created as backlog items. The existing-backlog-mapping.md recommended tagging 5 items for v004; all 5 were updated. Full coverage achieved.
