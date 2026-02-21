Investigate PR-001: "Session health: Orphaned WebFetch calls across 14 instances" — provide a detailed breakdown by version and session.

PR-001 reports 14 unique orphaned WebFetch tool_use calls found across 11 sessions. These HTTP requests were issued but never returned results, wasting execution time and blocking sessions.

## Investigation

Use the `query_cli_sessions` MCP tool to dig into the session analytics database. You'll need to query for orphaned WebFetch calls specifically.

1. **Version-by-version breakdown**: Which versions (v001-v007) had orphaned WebFetch calls? How many per version?
2. **Session-level detail**: For each affected session, identify: what theme/feature was being worked on, how many WebFetch calls orphaned, what URLs were being fetched (if available from session data)
3. **Timeline**: When did orphaned WebFetch calls start appearing? Was it concentrated in certain versions or spread evenly?
4. **Root causes**: Were these caused by: timeouts on slow URLs, fetching non-existent URLs, fetching URLs from memory vs search results, or something else?
5. **Impact**: Estimate total wasted time from orphaned WebFetch calls if the data supports it
6. **Current status**: Note that BL-054 (WebFetch safety rules in AGENTS.md) has been cancelled in favour of auto-dev-mcp BL-536 which addresses this at the MCP server level

Use query_cli_sessions with mode="analytics" or mode="raw" SQL as needed. Try query_type="orphaned" or search for "WebFetch" in tool_name filtering.

## Output Requirements

Create findings in comms/outbox/exploration/pr001-webfetch-breakdown/:

### README.md (required)
First paragraph: Summary of orphaned WebFetch distribution across versions.
Then: Key patterns, worst-affected versions, and whether the problem is getting better or worse.

### version-breakdown.md
Per-version table with: version, session count, orphaned WebFetch count, affected themes/features, and any identifiable root causes.

## Guidelines
- Under 200 lines per document
- Include actual session IDs and timestamps where available
- If query_cli_sessions doesn't have the granularity needed, say so explicitly
- Commit when complete

## When Complete
git add comms/outbox/exploration/pr001-webfetch-breakdown/
git commit -m "exploration: pr001-webfetch-breakdown complete"
