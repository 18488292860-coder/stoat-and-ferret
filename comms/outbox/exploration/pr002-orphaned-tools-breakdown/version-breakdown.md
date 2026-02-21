# Version Breakdown: Orphaned Non-WebFetch Tool Calls

## Note on Scope

The session analytics database returns sessions across all projects. Of 28 affected sessions, only 4 map to stoat-and-ferret. The remainder are auto-dev-mcp, ai-framework/Yocova, and cross-project sessions. All git_branch values are null, so version mapping relies on task descriptions and file paths.

## Per-Project Summary

| Project | Sessions | Orphaned Calls | Tool Types |
|---------|----------|---------------|------------|
| stoat-and-ferret | 4 | 5 | Task(1), TaskOutput(1), Read(3) |
| auto-dev-mcp | 8 | 10 | Write(1), Task(1), Bash(3), TaskOutput(3), Task(2) |
| ai-framework/Yocova | 7 | 8 | Grep(1), Task(3), Bash(2), Edit(1), Read(1) |
| auto-dev-test-target-1 | 1 | 1 | Bash(1) |
| Other/cross-project | 8 | 11 | Bash(5), TaskOutput(4), Read(2) |
| **Total** | **28** | **35** | |

## stoat-and-ferret Sessions

| Session ID | Date | Version | Orphans | Tools | Context |
|-----------|------|---------|---------|-------|---------|
| `2b3751e4` | Feb 8 | v004 | 1 | TaskOutput | design-v004 exploration; blocked polling subagent (310s timeout) |
| `832998f8` | Feb 13 | v006 | 1 | Task | design-v006-004-research; spawning FFmpeg expressions research agent |
| `agent-a2d8a40` | Feb 19 | — | 2 | Read(2) | Subagent reading C4 docs (gui-stores.md, gui-components-tests.md) |
| `agent-a0fa358` | Feb 19 | — | 1 | Read | Subagent reading api/app.py |

**Impact**: All stoat-and-ferret orphans are read-only operations (Read, TaskOutput polling) except for one Task spawn. No data loss risk.

## auto-dev-mcp Sessions

| Session ID | Date | Orphans | Tools | Context |
|-----------|------|---------|-------|---------|
| `dbaca306` | Jan 22 | 1 | Write | Writing retrospective-meta-analysis README.md |
| `bd143c2d` | Feb 1 | 1 | Task | Spawning "Create Theme 3 feature documents" agent |
| `agent-a5e391f` | Feb 1 | 1 | Bash | Subagent running mkdir for theme directories |
| `240a0257` | Feb 6 | 3 | TaskOutput(3) | C4-architecture: waiting on 3 parallel subagents (180s timeouts) |
| `1d497037` | Feb 6 | 1 | Bash | v050-retrospective: `sleep 120` waiting for quality gates |
| `5cc0ac9c` | Feb 6 | 1 | Bash | v050-retro-004-quality: running `uv run pytest -v` |
| `ee7dda9e` | Feb 14 | 2 | Task(2) | Launching 2 parallel research agents (WebFetch timeout, parallel streaming) |
| `dd1d5578` | Feb 21 | 1 | Bash | Running `uv run pytest` |

**Impact**: 1 Write orphan (exploration README) is the only potential data loss. Rest are blocking waits and subagent launches.

## ai-framework/Yocova Sessions

| Session ID | Date | Orphans | Tools | Context |
|-----------|------|---------|-------|---------|
| `b3acc3e9` | Feb 3 | 1 | Grep | Yocova UAT audit: searching SupportCenter_Ctrl.cls |
| `0bcb8587` | Feb 13 | 1 | Task | Spawning "Count fields per Yocova object" explorer |
| `agent-af225e7` | Feb 13 | 1 | Bash | Subagent: `grep -c` counting custom fields in .object file |
| `aaefbad2` | Feb 13 | 3 | Task(2), Edit(1) | CDC analysis: 2 subagent spawns + edit to generate_mapping.py |
| `08f1f0da` | Feb 17 | 1 | Read | Reading analysis-pass-10 README.md |
| `9798eba0` | Feb 19 | 2 | Bash, Read | Data mapping: Python extraction script + reading tool-results |
| `de65edd6` | Feb 20 | 1 | TaskOutput | analysis-pass-12: polling subagent (60s timeout) |

**Impact**: 1 Edit orphan to generate_mapping.py is the highest-risk item across all sessions.

## Other/Cross-Project Sessions

| Session ID | Date | Orphans | Tools | Context |
|-----------|------|---------|-------|---------|
| `99f08e8c` | Feb 5 | 1 | Bash | auto-dev-test-target-1: `sleep 300` |
| `52cf738c` | Feb 9 | 1 | TaskOutput | "list directories" session, polling subagent |
| `f9efa171` | Feb 12 | 1 | TaskOutput | Interrupted by user |
| `2a941a6d` | Feb 14 | 1 | Read | Reading gw-test implementation.md |
| `03679aa7` | Feb 16 | 1 | TaskOutput | Polling subagent (600s timeout) |
| `4501a34b` | Feb 18 | 1 | Bash | `sleep 90` waiting for task poll |
| `a56c2e2e` | Feb 18 | 1 | Bash | `sleep 60` checking test suite progress |
| `497080ec` | Feb 19 | 1 | Bash | `sleep 240` waiting for task poll |
| `6a2e8a73` | Feb 21 | 1 | Bash | `sleep 120` waiting for task poll |

## Timeline Distribution

| Week | Sessions | Orphans | Dominant Pattern |
|------|----------|---------|-----------------|
| Jan 20-26 | 1 | 1 | Write (exploration output) |
| Jan 27-Feb 2 | 2 | 2 | Task spawning |
| Feb 3-9 | 5 | 8 | TaskOutput polling (c4-arch session = 3 alone) |
| Feb 10-16 | 5 | 6 | Task spawning, user interruption |
| Feb 17-23 | 10 | 13 | Bash sleep/wait commands (8 of 13) |
