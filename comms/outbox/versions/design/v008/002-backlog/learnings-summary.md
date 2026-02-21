# Learnings Summary — v008 Design

Relevant learnings from the project learning repository, filtered by applicability to v008 scope (startup wiring, CI stability).

## Directly Applicable

### LRN-033: Fix CI Reliability Before Dependent Development Cycles
- **Tags:** process, ci, reliability, planning, failure-mode
- **Insight:** A single flaky test can block multiple features and themes. Fix CI reliability issues before starting development cycles that depend on CI for PR merges.
- **v008 applicability:** Directly motivates BL-055 (flaky E2E fix) as a P0 priority. v008 should ensure BL-055 is resolved before any subsequent versions add GUI features.

### LRN-031: Detailed Design Specifications Correlate with First-Iteration Success
- **Tags:** process, design, requirements, planning, quality
- **Insight:** Detailed specs with specific ACs, implementation plans, and scope boundaries lead to first-iteration success with no rework.
- **v008 applicability:** All 4 backlog items follow the current-state/gap/impact format with specific, testable ACs. This positions v008 for clean first-iteration execution.

### LRN-016: Validate Acceptance Criteria Against Codebase During Design
- **Tags:** process, design, requirements, validation
- **Insight:** Validate that ACs reference existing domain models and APIs during design to prevent unachievable requirements.
- **v008 applicability:** Critical for all items. ACs reference specific functions (`configure_logging()`, `create_tables()`), files (`ws.py`), and settings fields (`debug`, `ws_heartbeat_interval`). Must verify these exist and match descriptions before design.

### LRN-019: Build Infrastructure First in Sequential Version Planning
- **Tags:** process, planning, architecture, sequencing
- **Insight:** Sequence infrastructure themes before consumer themes.
- **v008 applicability:** Within Theme 1, database startup (BL-058) is infrastructure that should precede logging startup (BL-056), which may emit during DB initialization.

### LRN-029: Conscious Simplicity with Documented Upgrade Paths
- **Tags:** decision-framework, architecture, yagni, simplicity, trade-offs
- **Insight:** Choose the simplest implementation for current scope but document explicit upgrade triggers.
- **v008 applicability:** BL-058 AC #2 offers a choice between `create_tables()` and full Alembic migration at startup. The simpler option (create_tables) may suffice with a documented threshold for switching to Alembic.

## Contextually Relevant

### LRN-015: Single-Matrix CI Jobs for Expensive Operations
- **Tags:** ci, performance, process, github-actions
- **Insight:** Run expensive CI operations on a single platform rather than across the full test matrix.
- **v008 applicability:** If BL-055 fix involves changes to CI workflow configuration, this learning informs where E2E tests should run in the matrix.

### LRN-009: Handler Registration Pattern for Generic Job Queues
- **Tags:** architecture, patterns, job-queue, dependency-injection
- **Insight:** Explicit handler registration with factory-based DI provides clean, testable routing.
- **v008 applicability:** Low — informational context for understanding the existing DI pattern that BL-056/BL-058 wiring should follow.

### LRN-035: Automate Manual Maintenance When Third Instance Appears
- **Tags:** process, tooling, maintenance, automation, decision-framework
- **Insight:** Apply the rule of three to manual maintenance tasks.
- **v008 applicability:** v008 addresses the third discovered wiring gap (after logging and database, settings is the third). Consider whether a startup wiring checklist or automated verification would prevent future gaps.

## Not Applicable to v008

The following learnings were reviewed and found not relevant:
- LRN-010 (asyncio.Queue): No queue work in v008
- LRN-011 (Python/Rust boundaries): No Rust changes in v008
- LRN-028 (Implementation templates): v008 items are unique fixes, not a feature series
- LRN-030 (Architecture docs as feature): v008 scope is too small for dedicated doc features
- LRN-032 (Schema-driven architecture): No frontend schema work in v008
- LRN-034 (Validate numeric requirements): No numeric claims in v008 scope
- LRN-037 (Independent store interfaces): No frontend component work in v008
- LRN-038 (Sub-agent read-before-write): Tooling concern, not design concern
- LRN-039 (Context compaction): Session health concern, not design concern
