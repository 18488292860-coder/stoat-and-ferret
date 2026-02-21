# PR-002: Orphaned Non-WebFetch Tool Calls Analysis

Across 28 terminated sessions, 35 unique orphaned non-WebFetch tool calls were detected spanning 7 tool types: Bash (11), TaskOutput (8), Task (7), Read (6), Edit (1), Grep (1), Write (1). The orphaned calls span multiple projects (auto-dev-mcp, stoat-and-ferret, ai-framework, and others) from Jan 22 to Feb 21, 2026. Only 4 of the 28 sessions are directly attributable to stoat-and-ferret work. The dominant pattern is session termination during blocking waits — sleep commands, test suites, and TaskOutput polling account for 24 of 35 orphans (69%).

## Key Patterns

### 1. Blocking Waits Are the Primary Trigger (69%)
- **Sleep/wait commands**: 5 of 11 Bash orphans are `sleep` calls (90-300 seconds) used to poll long-running tasks
- **TaskOutput polling**: All 8 orphaned TaskOutput calls are blocking waits with 60-600s timeouts
- **Test suite runs**: 2 Bash orphans are `uv run pytest` commands, and 1 is a `sleep` waiting for quality gates
- Sessions hit timeouts or were manually terminated while blocked on these operations

### 2. Task Spawning at Session End (7 orphans)
All 7 orphaned Task calls are subagent launches (general-purpose or Explore agents). The parent session ended immediately after dispatching the task, before receiving the spawned agent's result. In 2 cases (session ee7dda9e), two Tasks were launched in parallel at the session's final moment.

### 3. No WebFetch Correlation
Zero of the 28 affected sessions have orphaned WebFetch calls. This is the opposite of PR-001's findings and indicates these are **independent issues** — the non-WebFetch orphans are not caused by the same WebFetch hang problem. The root causes are different: WebFetch orphans come from HTTP timeouts; non-WebFetch orphans come from session-level termination during blocking operations.

### 4. Duplicate Rows in Database
The raw (non-deduplicated) query returned 102 rows for 35 unique tool_use_ids, a ~3x inflation ratio. This is a data quality note for the ingestion pipeline — some tool calls appear duplicated across multiple ingestion passes.

## Lost Work Assessment

### Low Risk (33 of 35 orphans)
- **Read (6)**, **Grep (1)**: Non-destructive operations. No data loss possible.
- **Bash (11)**: Mostly sleep/wait or read-only commands. The `mkdir` command (agent-a5e391f) likely completed before the session recorded the result. No destructive operations.
- **TaskOutput (8)**: These are result-polling calls. The subagent may have completed its work independently; only the result retrieval was lost.
- **Task (7)**: Subagent launches. The subagent sessions may have run to completion independently.

### Moderate Risk (2 of 35 orphans)
- **Write** (toolu_018NPeNqqDbqjYMrnfpG3Tn7): Writing exploration README.md for auto-dev-mcp. The file content was documentation, not source code. If the write didn't complete, the exploration output would be missing or truncated.
- **Edit** (toolu_01VsYWsaZJy1bKcR1ptjz1fz): Editing `generate_mapping.py` in analysis-pass-7-data-mapping. This was modifying the `load_csv_data()` function. If the edit was partially applied, the file could be in an inconsistent state. However, this is a data analysis script, not production code.

## Systemic Causes

1. **Session timeout during blocking operations**: The most common cause. Sessions hit their maximum duration (or were manually stopped) while waiting on sleep commands, test suites, or subagent results. This is expected behavior for long-running sessions.

2. **User interruption**: At least one session (f9efa171) was explicitly "interrupted by user for tool use." Normal workflow.

3. **Subagent lifecycle mismatch**: When a parent session terminates while spawning or polling a subagent, both the Task launch and TaskOutput poll are recorded as orphaned, even though the subagent may complete independently.

## Recommendations

1. **No action required for data integrity** — the orphaned calls represent normal session termination patterns, not bugs or data corruption.
2. **Database deduplication** — the 3x row inflation from duplicate tool_use_ids should be investigated in the ingestion pipeline.
3. **For future analysis**, filter orphaned calls by tool type: only Edit/Write orphans represent potential data loss risk; Read/Grep/Bash(sleep)/TaskOutput are noise.
