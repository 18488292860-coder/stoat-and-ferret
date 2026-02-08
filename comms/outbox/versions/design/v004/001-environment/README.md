# Environment Verification - v004

**Status: READY.** The MCP server is healthy, the project is properly configured, git is on `main` and synced with remote, and version context is clear. No blockers for v004 design.

## Environment Status

- **MCP Server:** Healthy (v6.0.0, uptime 2492s)
- **Services:** config=ok, state=ok, execution=ok
- **External Dependencies:** git available, gh CLI available and authenticated
- **Source Sync:** Checksums match (running code matches source)
- **Tool Authorization:** Enabled, 4 active keys, no orphaned keys
- **Issues Found:** None

## Project Status

- **Project:** stoat-and-ferret (AI-driven video editor, Python/Rust hybrid)
- **Path:** C:/Users/grant/Documents/projects/stoat-and-ferret
- **Active Theme:** None (clean slate for v004)
- **Completed Themes:** 0 (in current session; v001-v003 completed historically)
- **Destructive Test Target:** false
- **Execution Backend:** legacy mode

## Git Status

- **Branch:** main (tracking origin/main)
- **Ahead/Behind:** 0/0 (fully synced)
- **Working Tree:** Not clean — 1 modified file:
  - `comms/state/explorations/design-v004-001-environment-1770582228447.json` (MCP exploration state file, expected — tracks this exploration)
- **Remote:** https://github.com/gwickman/stoat-and-ferret.git

## Architecture Context

- **C4 Documentation:** Does not exist yet. `docs/C4-Documentation/` directory not found.
- **BL-018 (P2):** "Create C4 architecture documentation" is a version-agnostic backlog item that can be addressed opportunistically.
- **Design Documents:** 8 authoritative design specs exist in `docs/design/` covering roadmap, architecture, prototype design, tech stack, API specification, risk assessment, quality architecture, and GUI architecture.
- **Key Architectural Patterns:** Non-destructive editing, Rust for compute (filters, timeline math, sanitization), Python for orchestration (FastAPI, routing), PyO3 bindings, transparency (FFmpeg filter strings in API responses).

## Version Scope

**v004 — Testing Infrastructure & Quality Verification** (Milestones M1.8–M1.9)

**Goal:** Black box testing harness, recording test doubles, contract tests, quality verification suite.

**Scope:** 8 new items + 5 existing items retagged = 13 backlog items total.

**New Items:** BL-020 through BL-027 (InMemory test doubles, DI for create_app, fixture factory, black box test catalog, contract tests with real FFmpeg, security audit, performance benchmark, async job queue).

**Retagged Items:** BL-009, BL-010, BL-012, BL-014, BL-016.

**Dependencies:** BL-021→BL-020, BL-022→BL-021, BL-023→BL-022, BL-027→BL-020.

**Deferred Items Arriving in v004:**
- Contract tests with real FFmpeg (deferred from M1.3/v001)
- Unify InMemory vs FTS5 search behavior (deferred from v002)
- Async scan job queue (deferred from v003)

**Constraints:** v004 is a prerequisite for v005 (black box tests validate GUI backend).
