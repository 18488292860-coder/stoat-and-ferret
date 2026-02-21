Investigate PR-002: "Session health: 34 orphaned non-WebFetch tool calls detected" — provide a detailed breakdown by version, tool type, and session.

PR-002 reports 34 unique orphaned tool calls (tool_use without tool_result) across 7 tool types: Bash (11), Task (7), TaskOutput (7), Read (6), Edit (1), Grep (1), Write (1). Orphaned Read/Edit/Write calls indicate abnormal session termination where work may have been lost.

## Investigation Steps

Use the `query_cli_sessions` MCP tool (with autoDevToolKey: 07a9ddeb-0ce3-4899-a8d3-0c90e052a20e) to dig into the session analytics database.

### Step 1: Get all orphaned non-WebFetch tool calls
Run this SQL query:
```sql
SELECT tc.timestamp, tc.tool_name, tc.tool_use_id, tc.bash_command, tc.bash_description, tc.task_description, 
       s.session_id, s.git_branch, s.started_at, s.is_subagent, s.agent_slug,
       ROUND(s.duration_secs / 60.0, 1) AS session_mins,
       SUBSTR(s.first_user_message, 1, 200) AS task,
       SUBSTR(tc.input_json, 1, 500) AS input_preview
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE
  AND s.ended_at IS NOT NULL
  AND tc.tool_name != 'WebFetch'
ORDER BY tc.timestamp
```

### Step 2: Get summary counts by tool type
```sql
SELECT tc.tool_name, COUNT(*) AS orphaned_count
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL AND tc.tool_name != 'WebFetch'
GROUP BY tc.tool_name
ORDER BY orphaned_count DESC
```

### Step 3: Get session-level summary for affected sessions
```sql
SELECT s.session_id, s.git_branch, s.started_at, s.ended_at,
       ROUND(s.duration_secs / 60.0, 1) AS session_mins,
       s.message_count, s.tool_call_count, s.error_count,
       s.is_subagent, s.agent_slug, s.parent_session_id,
       SUBSTR(s.first_user_message, 1, 200) AS task,
       COUNT(tc.id) AS orphaned_non_wf
FROM sessions s
JOIN tool_calls tc ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND tc.tool_name != 'WebFetch' AND s.ended_at IS NOT NULL
GROUP BY s.session_id
ORDER BY s.started_at
```

### Step 4: Check for WebFetch correlation
For each session that has orphaned non-WebFetch calls, check if it ALSO has orphaned WebFetch calls:
```sql
SELECT s.session_id, s.git_branch,
       SUM(CASE WHEN tc.tool_name != 'WebFetch' THEN 1 ELSE 0 END) AS orphaned_non_wf,
       SUM(CASE WHEN tc.tool_name = 'WebFetch' THEN 1 ELSE 0 END) AS orphaned_wf
FROM sessions s
JOIN tool_calls tc ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL
GROUP BY s.session_id
HAVING orphaned_non_wf > 0
ORDER BY s.started_at
```

### Step 5: Map sessions to versions
The project uses version folders like v001, v002, etc. The git_branch or first_user_message may contain version references. Also check the branch naming pattern. Run:
```sql
SELECT DISTINCT s.git_branch, s.started_at, SUBSTR(s.first_user_message, 1, 300) AS task
FROM sessions s
JOIN tool_calls tc ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND tc.tool_name != 'WebFetch' AND s.ended_at IS NOT NULL
ORDER BY s.started_at
```

### Step 6: Analyze Bash orphaned calls specifically
```sql
SELECT tc.timestamp, tc.bash_command, tc.bash_description, tc.elapsed_ms,
       s.session_id, s.git_branch
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL AND tc.tool_name = 'Bash'
ORDER BY tc.timestamp
```

### Step 7: Analyze Read orphaned calls
```sql
SELECT tc.timestamp, SUBSTR(tc.input_json, 1, 500) AS input_preview,
       s.session_id, s.git_branch
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL AND tc.tool_name = 'Read'
ORDER BY tc.timestamp
```

### Step 8: Analyze Edit/Write orphaned calls
```sql
SELECT tc.timestamp, tc.tool_name, SUBSTR(tc.input_json, 1, 500) AS input_preview,
       s.session_id, s.git_branch
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL AND tc.tool_name IN ('Edit', 'Write')
ORDER BY tc.timestamp
```

### Step 9: Analyze Task/TaskOutput orphaned calls
```sql
SELECT tc.timestamp, tc.tool_name, tc.task_description, SUBSTR(tc.input_json, 1, 500) AS input_preview,
       s.session_id, s.git_branch, s.is_subagent
FROM tool_calls tc
JOIN sessions s ON tc.session_id = s.session_id
WHERE tc.has_result = FALSE AND s.ended_at IS NOT NULL AND tc.tool_name IN ('Task', 'TaskOutput')
ORDER BY tc.timestamp
```

## Analysis Requirements

After gathering data from all queries above, analyze:

1. **Version-by-version breakdown**: Which versions (v001-v007) had orphaned non-WebFetch tool calls? How many per version, broken down by tool type? Map sessions to versions using git_branch or first_user_message content.

2. **Session-level detail**: For each affected session, identify: what theme/feature was being worked on, which tools orphaned, and whether the session terminated abnormally.

3. **Tool type analysis**: 
   - Bash (11): Were these long-running commands? What commands were being run?
   - Task/TaskOutput (7 each): Are these paired — i.e., tasks started but output never collected?
   - Read (6): What files were being read when the session died?
   - Edit/Write (1 each): Was work lost? Were these mid-file-modification?

4. **Pattern identification**: Are orphaned calls clustered around specific operations (e.g., CI waits, large file reads, complex git operations)?

5. **Impact assessment**: Estimate whether orphaned Edit/Write calls resulted in lost work or corrupted state.

6. **Correlation with WebFetch**: Do sessions with orphaned non-WebFetch calls also have orphaned WebFetch calls (PR-001), suggesting systematic session termination issues rather than tool-specific problems?

## Output Requirements

Create findings in comms/outbox/exploration/pr002-orphaned-tools-analysis/:

### README.md (required)
First paragraph: Summary of orphaned tool call distribution across versions and tool types.
Then: Key patterns, whether this represents lost work or just noisy session termination, and any systemic causes.

### version-breakdown.md
Per-version table with: version, session count, orphaned calls by tool type, affected themes/features, and identified patterns.

### tool-analysis.md
Per-tool-type analysis: what operations were interrupted, whether work was lost, and any common triggers.

## Guidelines
- Under 200 lines per document
- Include actual session IDs and timestamps where available
- If query_cli_sessions doesn't have the granularity needed, say so explicitly

## When Complete
git add comms/outbox/exploration/pr002-orphaned-tools-analysis/
git commit -m "exploration: pr002-orphaned-tools-analysis complete"