Read AGENTS.md first and follow all instructions there.

## Objective

Research unknowns, investigate prior art, and gather evidence for design decisions that affect version STRUCTURE (not implementation details) for stoat-and-ferret version v005.

## Context

This is Phase 1 (Environment & Investigation) for stoat-and-ferret version v005. Tasks 001-003 gathered context. This task resolves unknowns before proposing solutions.

v005 scope: GUI shell + library browser + project manager. Key areas: Frontend framework selection (Vite/React/etc.), WebSocket integration, static file serving from FastAPI, thumbnail generation pipeline, E2E testing infrastructure.

## Tasks

### 1. Identify Research Questions

Based on backlog items from Task 002 (read from `comms/outbox/versions/design/v005/002-backlog/backlog-details.md`), identify:
- Technologies or libraries requiring investigation
- Unclear areas of existing codebase
- Patterns needed but not yet understood
- Configuration values requiring evidence
- Integration points with existing systems

Key research areas for v005:
- Frontend framework selection (React vs Vue vs Svelte vs vanilla) and Vite setup
- FastAPI static file serving for frontend deployment (EXP-003 / BL-003)
- WebSocket implementation patterns with FastAPI
- Thumbnail generation approaches (FFmpeg frame extraction)
- E2E testing frameworks (Playwright vs Cypress vs similar)
- How the existing API structure supports WebSocket integration

### 2. Codebase Investigation

For questions about existing code:
- Use `request_clarification` to search for patterns, classes, or files
- Read relevant source files to understand current implementation
- Focus on: existing API structure, route patterns, middleware, static file config
- Document findings with file paths and line references

### 3. Verify Referenced Learnings

If any backlog items reference learnings (LRN-XXX):
1. Read the learning
2. Verify the condition still exists
3. Mark as STALE or VERIFIED with evidence

### 4. External Research

Use DeepWiki MCP tools to research:
- FastAPI WebSocket best practices (ask_question on repo "tiangolo/fastapi")
- Vite project setup patterns (ask_question on repo "vitejs/vite")
- Playwright E2E testing patterns (ask_question on repo "microsoft/playwright")
- React + Vite integration patterns

### 5. Evidence Gathering for Concrete Values

For any concrete values needed (WebSocket heartbeat intervals, thumbnail dimensions, etc.):
- Query existing codebase for current values
- Check configuration files
- If not determinable, mark as "TBD - requires runtime testing"

### 6. Impact Analysis

Document:
- **Dependencies**: What existing code/tools/configs will change?
- **Breaking changes**: Will existing workflows or APIs break?
- **Test impact**: What tests need creation or update?
- **Documentation**: What docs require update after implementation?

## Output Requirements

Save ALL outputs to BOTH locations:

**Primary (exploration output):** `comms/outbox/exploration/design-v005-004-research/`
**Design artifact store:** `comms/outbox/versions/design/v005/004-research/`

Write identical files to both locations.

### README.md (required)

First paragraph: Summary of research scope and key findings.

Then:
- **Research Questions**: List of questions investigated
- **Findings Summary**: High-level results
- **Unresolved Questions**: Items that cannot be resolved pre-implementation
- **Recommendations**: Design direction based on research

### codebase-patterns.md

Findings from codebase investigation:
- Existing patterns found (API routing, middleware, DI)
- File paths and key classes/functions
- Code snippets showing relevant implementations

### external-research.md

Findings from DeepWiki and web research:
- Libraries/tools evaluated
- Patterns and best practices found
- Sources cited (with URLs or repo names)
- Recommended approaches with rationale

### evidence-log.md

For all concrete values needed:
- Parameter name, value, source, data, rationale

### impact-analysis.md

Analysis of implementation impact:
- Dependencies (code/tools/configs affected)
- Breaking changes identified
- Test infrastructure needs
- Documentation updates required

## Allowed MCP Tools

- `request_clarification`
- `read_document`
- `start_exploration`
- `get_exploration_status`
- `get_exploration_result`
- `list_explorations`
- `get_learning`
- `search_learnings`

## Guidelines

- ALL backlog items from PLAN.md are MANDATORY — research must cover all of them
- Document sources for all findings (file paths, URLs, repo names)
- Mark items as "TBD - requires [X]" only if truly not determinable pre-implementation
- Never defer research to Claude Code with phrases like "Check if..." or "Investigate..."
- Keep each document under 200 lines
- Do NOT commit — the orchestrator handles commits after Phase 2

Do NOT commit or push — the calling prompt handles commits. Results folder: design-v005-004-research.