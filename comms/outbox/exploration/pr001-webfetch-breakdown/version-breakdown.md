# Orphaned WebFetch: Version & Project Breakdown

## Stoat-and-Ferret by Version

| Version | Dates | Orphaned | Sessions | Theme/Feature | URLs |
|---------|-------|----------|----------|---------------|------|
| v001 | Jan 26 | 0 | 0 | - | - |
| v002 | Jan 26-27 | 2 | 2 | 01-rust-python-bindings / 001-stub-regeneration | pyo3-stub-gen GitHub repo |
| v003 | Jan 28 | 0 | 0 | - | - |
| v004 | Feb 8-9 | 0 | 0 | - | - |
| v005 | Feb 9-10 | 0 | 0 | - | - |
| v006 | Feb 13-19 | 3 | 2 | Design research (exploration) | FFmpeg docs/blogs |
| v007 | Feb 19 | 0 | 0 | - | - |

**Total stoat-and-ferret: 5 orphaned / 4 sessions / 2 versions**

## All 14 Orphaned Calls (Cross-Project)

| # | Date | Session | Project | Context | URL | Root Cause |
|---|------|---------|---------|---------|-----|------------|
| 1 | Jan 26 18:43 | `23c77755` | stoat-and-ferret | v002 stub-regen | github.com/Jij-Inc/pyo3-stub-gen/tree/main/examples | GitHub page (use gh CLI) |
| 2 | Jan 26 19:44 | `47054390` | stoat-and-ferret | v002 stub-regen | github.com/Jij-Inc/pyo3-stub-gen | GitHub page (use gh CLI) |
| 3 | Feb 09 21:08 | `agent-a73b721` | kb-store | Freelance search | www.upwork.com/freelance-jobs/apply/... | Auth-walled (login required) |
| 4 | Feb 09 21:38 | `agent-ab601f1` | kb-store | Bounty research | github.com/MonoGame/MonoGame/issues/8120 | GitHub page |
| 5 | Feb 09 21:38 | `agent-ab31cde` | kb-store | Bounty research | algora.io/golemcloud/home | Third-party platform |
| 6 | Feb 09 21:38 | `agent-a3ef471` | kb-store | Bounty research | github.com/golemcloud/golem-ai/issues/23 | GitHub page |
| 7 | Feb 13 22:10 | `agent-a8410c2` | stoat-and-ferret | v006 design research | ffmpegbyexample.com/examples/... | Third-party blog |
| 8 | Feb 13 22:10 | `agent-a8410c2` | stoat-and-ferret | v006 design research | www.braydenblackwell.com/blog/ffmpeg-text-rendering | Third-party blog |
| 9 | Feb 14 11:36 | `agent-ab353ea` | gw-test | CLI research | gist.github.com/tirufege/... | GitHub Gist |
| 10 | Feb 14 12:29 | `agent-ae168d6` | auto-dev-mcp | WebFetch research | platform.claude.com/docs/.../implement-tool-use | Anthropic docs (auth?) |
| 11 | Feb 14 12:30 | `4a029681` | auto-dev-mcp | WebFetch hang exploration | code.claude.com/docs/en/sub-agents | Anthropic docs (auth?) |
| 12 | Feb 14 12:31 | `agent-a5f8dfc` | auto-dev-mcp | WebFetch research | releasebot.io/updates/anthropic/claude-code | Third-party tracker |
| 13 | Feb 15 21:36 | `98c9d711` | auto-dev-mcp | v066 design research | platform.claude.com/docs/.../pricing | Anthropic docs (auth?) |
| 14 | Feb 18 14:31 | `792eb162` | stoat-and-ferret | v006 design research | ayosec.github.io/ffmpeg-filters-docs/.../atempo.html | GitHub Pages static site |

## Session Detail

### Stoat-and-Ferret Sessions

| Session | Duration | Tool Calls | Errors | Sub-agent? | Parent Context |
|---------|----------|------------|--------|------------|----------------|
| `23c77755` | 2.9 min | 56 | 4 | No | v002 execution prompt |
| `47054390` | 1.0 min | 144 | 18 | No | v002 execution prompt (retry) |
| `agent-a8410c2` | 1.2 min | 16 | 0 | Yes | Spawned by `832998f8` (v006 design research) |
| `792eb162` | 4.9 min | 144 | 0 | No | v006 design research exploration |

### Non-Stoat-and-Ferret Sessions

| Session | Duration | Project | Context |
|---------|----------|---------|---------|
| `agent-a73b721` | 0.6 min | kb-store | Upwork job search |
| `agent-ab601f1` | 0.8 min | kb-store | Multi-platform bounty research |
| `agent-ab31cde` | 1.0 min | kb-store | Algora bounty research |
| `agent-a3ef471` | 1.1 min | kb-store | Niche technology bounty research |
| `agent-ab353ea` | 2.7 min | gw-test | Claude Code CLI session research |
| `agent-ae168d6` | 0.6 min | auto-dev-mcp | Parallel tool execution research |
| `4a029681` | 2.4 min | auto-dev-mcp | WebFetch hang prevention exploration |
| `agent-a5f8dfc` | 2.5 min | auto-dev-mcp | WebFetch timeout research |
| `98c9d711` | 10.9 min | auto-dev-mcp | Design v066 research |

## URL Pattern Analysis

| Category | Count | URLs | Root Cause |
|----------|-------|------|------------|
| GitHub repos/issues | 5 | pyo3-stub-gen, MonoGame, golem-ai, gist | Should use `gh` CLI per WebFetch docs |
| Anthropic docs | 3 | platform.claude.com, code.claude.com | Likely auth-walled or anti-bot |
| Third-party blogs | 3 | ffmpegbyexample, braydenblackwell, releasebot.io | Anti-bot or slow response |
| Auth-walled platforms | 2 | upwork.com, algora.io | Login required |
| Static sites | 1 | ayosec.github.io (GitHub Pages) | Possible anti-bot or timeout |

## Data Limitations

- All sub-agent sessions have `parent_session_id = null` - the parent link was lost during JSONL ingestion
- Parent sessions were inferred by matching timestamps and project context
- `elapsed_ms` is null for all 14 orphaned calls, confirming no timing data was recorded before the hang
- The original health check used `scope=all`, which is why non-stoat-and-ferret sessions were included
