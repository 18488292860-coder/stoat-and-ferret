# Session Health Findings Detail - v008 Retrospective

## Pattern 1: Hung WebFetch Calls

### Query Used
```
query_cli_sessions(
    project="stoat-and-ferret",
    mode="troubleshoot",
    query_type="orphaned_tools"
)
```
Filtered results to WebFetch tool calls only.

### Raw Results (WebFetch only)

| Timestamp | Tool Use ID | Session ID | Session Ended | Input Preview |
|-----------|-------------|------------|---------------|---------------|
| 2026-02-18T14:31:15Z | `toolu_01QygWk7ruTJJ4wEMJ5z3bWk` | `792eb162-520c-44c4-b20c-7aa9535bd873` | 2026-02-18T14:31:33Z | URL: `https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Audio/atempo.html` |

Note: 4 duplicate rows returned for the same tool_use_id (ingestion pipeline duplication per PR-002).

### Classification Reasoning

- **Threshold:** Any orphaned WebFetch = HIGH (zero tolerance)
- **Match count:** 1 unique orphaned WebFetch call
- **Classification:** HIGH
- **Remediation status:** Already addressed by PR-001 (completed). Root cause: WebFetch has no built-in timeout. Global fix via auto-dev-mcp BL-536.

---

## Pattern 2: Tool Authorization Retry Waste

### Query Used
```sql
SELECT
    tc.session_id,
    tc.tool_name,
    COUNT(*) as denial_count
FROM tool_calls tc
WHERE tc.is_error = TRUE
  AND tc.result_snippet LIKE '%UNAUTHORIZED%'
GROUP BY tc.session_id, tc.tool_name
HAVING COUNT(*) >= 3
ORDER BY denial_count DESC
```

### Raw Results

| Session ID | Tool Name | Denial Count |
|------------|-----------|-------------|
| `5bf50e64-3af7-4542-8299-6c8a89c70d73` | Bash | 12 |

### Classification Reasoning

- **Threshold:** Same tool denied 3+ times in one session = MEDIUM
- **Match count:** 1 session
- **Classification:** MEDIUM
- **Detail:** Agent in session `5bf50e64-3af7-4542-8299-6c8a89c70d73` attempted Bash commands 12 times despite repeated UNAUTHORIZED errors. This suggests the agent did not adapt its approach after initial denials. Each retry wastes ~1 API round-trip.
- **Systemic assessment:** Isolated (1 of 3,123 sessions). No product request created.

---

## Pattern 3: Orphaned Tool Calls (Non-WebFetch)

### Query Used
```
query_cli_sessions(
    project="stoat-and-ferret",
    mode="troubleshoot",
    query_type="orphaned_tools"
)
```
Filtered results to exclude WebFetch tool calls.

### Raw Results (Non-WebFetch, deduplicated)

| Timestamp | Tool Use ID | Tool Name | Session ID | Session Ended | Input Preview |
|-----------|-------------|-----------|------------|---------------|---------------|
| 2026-02-19T22:05:18Z | `toolu_01Qz9GMp1sQKPfJQkBoqYNth` | Read | `agent-a0fa358` | 2026-02-19T22:10:53Z | `app.py` |
| 2026-02-19T21:58:27Z | `toolu_01WHWFtS8ptyqPPRpJ9BZqdX` | Read | `agent-a2d8a40` | 2026-02-19T22:03:49Z | `c4-code-gui-stores.md` |
| 2026-02-19T21:58:28Z | `toolu_016Lh84ob4CzuEoSiZtXTNgn` | Read | `agent-a2d8a40` | 2026-02-19T22:03:49Z | `c4-code-gui-components-tests.md` |
| 2026-02-18T16:20:10Z | `toolu_018LUWwLQhcD2oDbAPUrGzKK` | Bash | `4501a34b-449d-40f4-87ae-eebbb6f55a46` | 2026-02-18T16:20:10Z | `sleep 90` |

Note: Multiple duplicate rows per tool_use_id in raw output (ingestion pipeline duplication).

### Classification Reasoning

- **Threshold:** Any orphaned non-WebFetch tool call = HIGH (zero tolerance)
- **Match count:** 4 unique orphaned calls across 3 sessions
- **Classification:** HIGH
- **Tool breakdown:** Read (3), Bash (1)
- **Remediation status:** Already addressed by PR-002 (completed). Investigation found 69% of orphans are blocking waits during normal session termination. Lost work risk minimal.
- **Session detail:**
  - `agent-a0fa358` and `agent-a2d8a40`: Sub-agent sessions reading documentation files. Sessions ended normally; orphaned Reads indicate session termination during parallel file reads.
  - `4501a34b-...`: `sleep 90` command orphaned at exact session end time, indicating the session was terminated while waiting.

---

## Pattern 4: Excessive Context Compaction

### Query Used
```sql
SELECT
    ce.session_id,
    COUNT(*) as compaction_count
FROM compaction_events ce
GROUP BY ce.session_id
HAVING COUNT(*) >= 3
ORDER BY compaction_count DESC
```

### Raw Results

| Session ID | Compaction Count |
|------------|-----------------|
| `f9efa171-4e9b-4a13-9796-2fdbd353bedd` | 16 |
| `aaefbad2-2044-476a-8439-7b370cb813d4` | 12 |
| `247375f3-ba69-4cbd-b60e-f4aa43045131` | 12 |
| `b3acc3e9-2adf-46dd-934d-7bc2294c13ec` | 10 |
| `9b6aed8e-8f2c-461a-8cb1-94e0d6cb2a8c` | 10 |
| `de65edd6-89ec-45b3-ad67-cd27f55805b7` | 7 |
| `52cf738c-93ba-4068-9599-131dc0a71100` | 7 |
| `1093ba6f-b262-4adb-912c-890b4f13b4e1` | 7 |
| `7458fba6-90bb-4c0f-8cc0-b3fa32b66930` | 6 |
| `47b617ad-1e6a-492d-b6c7-ab631cd6e5fb` | 6 |
| `3f0bd0c9-d89e-4c9a-bcea-e229d77bc9c1` | 6 |
| `17b54bed-4830-4bee-a4fb-b7e02a647dca` | 6 |
| `0809e915-4edc-4308-b453-7ea17c5d4b3e` | 6 |
| `fe1c4c9f-fe62-400c-b804-fcb24148e32a` | 5 |
| `764b36bd-9181-4f8e-877a-24d1ead896e1` | 5 |
| `65c10871-a2c9-4488-9853-45b8e5d5f584` | 5 |
| `c61dc1f6-30ad-4542-a63c-b5e96135823a` | 4 |
| `9826ce4b-6b64-4961-bbfc-3e60ad2afd4f` | 4 |
| `9798eba0-1162-48fd-9a98-a9e05eb5bac0` | 4 |
| `7ee83eb8-ea60-4731-beed-a03833e67a9f` | 4 |
| `133419bc-b8f2-4ce7-8c8b-1a537429adde` | 4 |
| `fd64f530-4e71-4ad9-917e-503019a368c2` | 3 |
| `f17b818e-9aaa-4b59-bf25-8f75f8458b88` | 3 |
| `c8232369-2c44-4e5c-9a68-2dc5bdb540b1` | 3 |
| `agent-acdfa6c` | 3 |
| `9b738766-0aae-4e4c-a38b-d6768fea9e64` | 3 |
| `2a941a6d-d5e1-4d78-baf0-c25bc2d654e3` | 3 |

### Classification Reasoning

- **Threshold:** 3+ compaction events per session = MEDIUM
- **Match count:** 27 sessions
- **Classification:** MEDIUM
- **Distribution:** 5 sessions had 10+ compactions (extreme), 8 had 5-9 (high), 14 had 3-4 (moderate)
- **Percentage of total:** 27 of 3,123 sessions (0.9%)
- **Systemic assessment:** While the absolute count (27) is notable, it represents less than 1% of total sessions. The 5 sessions with 10+ compactions are concerning (possible loss of implementation context), but compaction is expected behavior for complex multi-file implementation tasks. Recommend monitoring this metric across retrospective cycles to detect trends.
- **No product request created** - below systemic threshold, but flagged for trend monitoring.

---

## Pattern 5: Sub-Agent Failure Cascades

### Query Used
```sql
SELECT
    s.session_id,
    s.duration_secs,
    s.parent_session_id,
    COALESCE(err.error_count, 0) as error_count,
    COALESCE(orph.orphaned_count, 0) as orphaned_count
FROM sessions s
LEFT JOIN (
    SELECT session_id, COUNT(*) as error_count
    FROM tool_calls
    WHERE is_error = TRUE
    GROUP BY session_id
) err ON s.session_id = err.session_id
LEFT JOIN (
    SELECT session_id, COUNT(*) as orphaned_count
    FROM tool_calls
    WHERE has_result = FALSE
    GROUP BY session_id
) orph ON s.session_id = orph.session_id
WHERE s.is_subagent = TRUE
  AND s.duration_secs > 1800
  AND (COALESCE(err.error_count, 0) > 0 OR COALESCE(orph.orphaned_count, 0) > 0)
ORDER BY s.duration_secs DESC
```

### Raw Results

| Session ID | Duration (secs) | Parent Session | Error Count | Orphaned Count |
|------------|----------------|----------------|-------------|----------------|
| `agent-a55770f` | 3,190 (~53 min) | null | 56 | 0 |

### Classification Reasoning

- **Threshold:** Sub-agents with duration >30 minutes + errors or orphaned tools
- **Severity rule:** HIGH if both error_count > 0 AND orphaned_count > 0; MEDIUM otherwise
- **Match count:** 1 sub-agent
- **Classification:** MEDIUM (errors only; orphaned_count = 0)
- **Detail:** Sub-agent `agent-a55770f` ran for 53 minutes with 56 tool call errors but no orphaned tools. The null parent_session_id may indicate the parent session data was not captured or the sub-agent was spawned differently. The 56 errors over 53 minutes (~1 error/minute) suggests repeated failures rather than a single cascade.
- **Systemic assessment:** Isolated (1 of 1,175 sub-agent sessions, 0.09%). No product request created.
