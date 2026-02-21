# PR-001: Orphaned WebFetch Detailed Breakdown

Of the 14 orphaned WebFetch calls identified in the v007 session health check, only **5 belong to stoat-and-ferret** (across 4 sessions in v002 and v006). The remaining 9 originated from other projects: kb-store (4), auto-dev-mcp (4), and gw-test (1). The original health check queried with `scope=all`, making the cross-project contamination invisible. Within stoat-and-ferret, orphaned WebFetch is concentrated in research/exploration tasks, not execution.

## Key Findings

### 1. Cross-Project Contamination

The original health check reported 14 orphaned WebFetch calls as a stoat-and-ferret problem. In reality:

| Project | Orphaned Calls | Sessions | Context |
|---------|---------------|----------|---------|
| stoat-and-ferret | 5 | 4 | v002 execution, v006 design research |
| kb-store | 4 | 4 | Bounty/freelance research (not dev work) |
| auto-dev-mcp | 4 | 4 | WebFetch hang investigation, design research |
| gw-test | 1 | 1 | CLI session folder research |

### 2. Stoat-and-Ferret Breakdown

Only 2 of 7 versions were affected:

- **v002** (Jan 26): 2 orphaned calls fetching `github.com/Jij-Inc/pyo3-stub-gen` URLs during stub-regeneration feature work
- **v006** (Feb 13-18): 3 orphaned calls fetching FFmpeg documentation URLs during design research explorations

v001, v003, v004, v005, and v007 had zero orphaned WebFetch calls.

### 3. Overall Rate

Across the entire database: **14 orphaned out of 223 unique WebFetch calls (6.3%)**. All 14 have `elapsed_ms = null`, confirming the tool hung indefinitely without returning.

### 4. Timeline

```
Jan 26  ██ .......... v002: 2 orphaned (pyo3-stub-gen GitHub URLs)
Feb 09  ████ ........ kb-store: 4 orphaned (bounty research, same day)
Feb 13  ███ ......... v006: 2 orphaned + 1 gw-test (FFmpeg + CLI research)
Feb 14  ███ ......... auto-dev-mcp: 3 orphaned (WebFetch investigation, ironic)
Feb 15  █ ........... auto-dev-mcp: 1 orphaned (design research)
Feb 18  █ ........... v006: 1 orphaned (FFmpeg atempo docs)
```

The problem is **not getting worse** - it appears whenever research sub-agents fetch URLs from web search results. Feb 9 was the worst single day (4 calls) but that was non-stoat-and-ferret work.

### 5. Root Cause Patterns

All 14 orphaned calls share a common profile: sub-agents or exploration sessions fetching URLs discovered via web search. URL categories:

- **GitHub pages** (5): `github.com` repos/issues/gists - WebFetch docs explicitly say to use `gh` CLI instead
- **Anthropic docs** (3): `platform.claude.com`, `code.claude.com` - may require authentication
- **Third-party blogs/platforms** (4): Various sites with possible anti-bot protections
- **Auth-walled pages** (2): Upwork (login required), Algora

The root cause is **WebFetch having no effective timeout** combined with agents fetching URLs that will never return clean content.

### 6. Impact

Low for stoat-and-ferret specifically. Affected sessions were short (1-5 minutes), suggesting sessions were killed quickly when WebFetch hung. No evidence of the 3-hour hang scenario seen elsewhere.

### 7. Current Status

BL-054 (WebFetch safety rules in AGENTS.md) has been cancelled in favour of **auto-dev-mcp BL-536**, which addresses orphaned WebFetch at the MCP server level rather than relying on agent-level prompt guardrails.
