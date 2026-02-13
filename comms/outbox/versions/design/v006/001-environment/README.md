# 001-Environment Verification — v006

Environment is **ready** for v006 design. All systems healthy, project configured, working tree nearly clean (only MCP state file modified), and comprehensive C4 architecture docs current as of v005. Version scope: 7 backlog items (BL-037–043) covering effects engine foundation — filter expression engine, graph validation, text overlay, and speed control.

## Environment Status

- **MCP Server:** Healthy (v6.0.0, uptime 18105s)
- **Services:** config=ok, state=ok, execution=ok
- **External Dependencies:** git available, gh CLI available and authenticated
- **Source Sync:** In sync (checksums match)
- **Tool Authorization:** Enabled, 2 active keys, no orphaned keys
- **No critical errors** reported

## Project Status

- **Project:** stoat-and-ferret — AI-driven video editor with hybrid Python/Rust architecture
- **Active Theme:** None
- **Completed Themes:** 0 (for v006; previous versions v001–v005 all complete)
- **Destructive Test Target:** false
- **Execution Backend:** legacy mode
- **Timeout:** 180 minutes

## Git Status

- **Branch:** main
- **Remote:** origin (https://github.com/gwickman/stoat-and-ferret.git)
- **Working Tree:** Nearly clean — one modified file: `comms/state/explorations/design-v006-001-env-1771019673679.json` (MCP exploration state, expected)
- **No untracked files, no staged changes**

## Architecture Context

- **C4 Documentation:** Complete, last updated 2026-02-10 for v005 (full generation)
- **Levels Documented:** Context, Container, Component, Code (46 documents total)
- **API Spec:** OpenAPI 3.1 at `docs/C4-Documentation/apis/api-server-api.yaml` (v0.3.0)
- **Key Architecture:** 3-tier (React GUI → FastAPI API → Rust core via PyO3), SQLite persistence, FFmpeg subprocess execution
- **Relevant for v006:** Rust core engine component (`c4-component-rust-core-engine.md`) covers timeline math, clip validation, FFmpeg command building — v006 extends this with filter expression engine and graph validation

## Version Scope

- **Version:** v006 — Effects Engine Foundation
- **Goal:** Greenfield Rust filter expression engine, graph validation, text overlay, speed control, effect discovery API
- **Roadmap Reference:** Phase 2, Milestones M2.1–2.3
- **Estimated Scope:** 7 backlog items

### Backlog Items

| ID | Priority | Title |
|----|----------|-------|
| BL-037 | P1 | Implement FFmpeg filter expression engine in Rust |
| BL-038 | P1 | Implement filter graph validation |
| BL-039 | P1 | Build filter composition system |
| BL-040 | P1 | Implement drawtext filter builder |
| BL-041 | P1 | Implement speed control filters |
| BL-042 | P2 | Create effect discovery API endpoint |
| BL-043 | P2 | Apply text overlay to clip API |

### Dependencies

- v006 is **independent of v005** (pure Rust + API work)
- BL-039 → BL-038 (composition needs validation)
- BL-040 → BL-037 (drawtext needs expression engine)
- BL-042 → BL-040 + BL-041 (discovery needs builders)
- BL-043 → BL-040 + BL-042 (clip API needs drawtext + discovery)
- BL-043 may need an exploration for clip effect model design (EXP BL-043, status: pending)

### Constraints

- BL-047 (Effect registry schema) is pending investigation, informing v007 — not blocking v006
- BL-051 (Preview thumbnail pipeline) also pending for v007
- Deferred from prior versions: Rust coverage threshold at 75% (target 90%), SPA fallback routing
