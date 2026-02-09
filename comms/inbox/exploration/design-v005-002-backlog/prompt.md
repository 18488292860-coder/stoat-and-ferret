Read AGENTS.md first and follow all instructions there.

## Objective

Gather complete details for all backlog items in v005 scope and learn from the previous version's retrospective for stoat-and-ferret.

## Context

This is Phase 1 (Environment & Investigation) for stoat-and-ferret version v005. Task 001 identified the backlog item IDs. This task gathers full details.

## Tasks

### 1. Fetch Complete Backlog Items

For each backlog ID in v005 scope, call `get_backlog_item(project="stoat-and-ferret", item_id="BL-XXX")`:
- BL-003 (EXP-003: FastAPI static file serving)
- BL-028 (EXP: Frontend framework selection and Vite setup)
- BL-029 (WebSocket endpoint for real-time events)
- BL-030 (Application shell and navigation)
- BL-031 (Dashboard panel)
- BL-032 (Thumbnail generation pipeline)
- BL-033 (Library browser)
- BL-034 (Fix pagination total count)
- BL-035 (Project manager)
- BL-036 (E2E test infrastructure)

Extract full details: title, description, acceptance criteria, priority, tags, notes.

**MANDATORY SCOPE:** ALL backlog items listed in PLAN.md for this version are mandatory. No items may be deferred.

### 2. Quality Assessment for Each Item

For each backlog item, assess quality across three dimensions:

#### 2a. Description Depth
- Count approximate words in the item's description
- Flag items with descriptions under ~50 words as potentially thin
- If thin, use `update_backlog_item` to expand with problem context and impact statement

#### 2b. Acceptance Criteria Testability
- For each acceptance criterion, check for the presence of an action verb
- Flag noun-phrase-only criteria as potentially non-testable
- If non-testable, use `update_backlog_item` to rewrite with specific, verifiable actions

#### 2c. Use Case Authenticity
- Check use_case text against known formulaic patterns
- Flag formulaic use cases that add no information beyond the title
- If formulaic, use `update_backlog_item` to write an authentic user scenario

#### 2d. Assessment Summary
Document assessment results for each item in the output.

### 3. Identify Previous Version

Call `list_versions(project="stoat-and-ferret")` to find the most recent completed version before v005 (should be v004).

### 4. Review Previous Retrospective

Locate and read the previous version's retrospective:
- Check `docs/versions/v004/README.md`
- Check `comms/outbox/v004/retrospective.md`
- Check `comms/outbox/versions/retrospective/v004/` for retrospective outputs

Extract insights relevant to v005 design:
- Learnings that apply to v005 scope
- Patterns that worked well
- Tech debt to address
- Recommendations for future work

### 5. Search Project Learnings

Run `search_learnings(project="stoat-and-ferret", tags=["design", "architecture"])` to find relevant documented learnings.

Review any learnings that might inform the design approach for v005.

## Output Requirements

Save ALL outputs to BOTH locations:

**Primary (exploration output):** `comms/outbox/exploration/design-v005-002-backlog/`
**Design artifact store:** `comms/outbox/versions/design/v005/002-backlog/`

Write identical files to both locations.

### README.md (required)

First paragraph: Summary of backlog scope and key insights from retrospective.

Then:
- **Backlog Overview**: Count of items, priority distribution
- **Previous Version**: Version number, retrospective location
- **Key Learnings**: Top insights applicable to current version
- **Tech Debt**: Any debt items to address
- **Missing Items**: Any referenced backlog items not found

### backlog-details.md

For each backlog item, include:
- ID, Title, Priority
- Full description (quoted)
- Acceptance criteria (listed)
- Tags and notes
- Quality assessment results
- Complexity assessment (based on description, not title)

### retrospective-insights.md

Synthesized insights from previous version:
- What worked well (to continue)
- What didn't work (to avoid)
- Tech debt addressed vs deferred
- Architectural decisions to inform current version

### learnings-summary.md

Relevant learnings from project learning repository:
- Learning ID and title
- Summary of insight
- Applicability to current version

## Allowed MCP Tools

- `get_backlog_item`
- `update_backlog_item`
- `list_versions`
- `read_document`
- `search_learnings`
- `list_learnings`
- `get_learning`

## Guidelines

- ALL backlog items from PLAN.md are MANDATORY — no deferrals, no descoping
- Quote actual backlog text when documenting items
- Distinguish title complexity from implementation complexity
- Connect retrospective insights to current backlog items where possible
- Keep documents under 200 lines each
- Do NOT commit — the orchestrator handles commits after Phase 2

Do NOT commit or push — the calling prompt handles commits. Results folder: design-v005-002-backlog.