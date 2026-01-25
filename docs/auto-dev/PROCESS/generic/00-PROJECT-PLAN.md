# Project Planning Process

## Purpose

Create and maintain `docs/auto-dev/PLAN.md` — the bridge between strategic vision and tactical execution. This document maps high-level roadmap goals to auto-dev versions, tracks investigation dependencies, and records scoping decisions.

## When to Use

- **Project bootstrap**: Create PLAN.md when initializing auto-dev for a new project
- **Version planning**: Update before scoping each new version
- **Post-retrospective**: Update after completing versions to reflect learnings
- **Scope changes**: Update when strategic priorities shift

## Inputs

- **Strategic roadmap** — The project's long-term vision document (location varies by project)
- **Backlog** — `docs/auto-dev/backlog.json` for tactical work items
- **Explorations** — Results from `comms/outbox/exploration/` investigations
- **Learnings** — `docs/auto-dev/LEARNINGS/` captured knowledge
- **Version retrospectives** — `docs/versions/vXXX/` completion summaries

## The Integration Model

PLAN.md connects five artifacts:

| Artifact | Role | Location |
|----------|------|----------|
| Strategic Roadmap | Vision, phases, milestones | Project-specific (e.g., `docs/design/roadmap.md`) |
| PLAN.md | Roadmap → version mapping | `docs/auto-dev/PLAN.md` |
| Backlog | Tactical parking lot | `docs/auto-dev/backlog.json` |
| Explorations | Pre-design research | `comms/outbox/exploration/` |
| Learnings | Post-execution knowledge | `docs/auto-dev/LEARNINGS/` |

**Flow:**
```
Strategic Roadmap
       ↓
   PLAN.md ←──────── Explorations (reduce uncertainty)
       ↓                    ↑
   Versions ──────────→ Learnings (inform future)
       ↓
   Backlog (escape valve for discovered work)
```

## Process

### 1. Create Initial PLAN.md

At project bootstrap, create the document with:
- Link to strategic roadmap
- Initial version mapping (rough, will refine)
- Known investigation needs

### 2. Identify Investigation Dependencies

Before designing a version, determine what unknowns need resolution:

- **Technical spikes**: "How does X library handle Y?"
- **Architecture decisions**: "Which pattern fits our constraints?"
- **Integration questions**: "What's the actual API shape?"

Create exploration tasks for each. Record in PLAN.md with:
- Investigation ID (e.g., `EXP-001`)
- Question being answered
- Which version(s) it informs

### 3. Map Roadmap to Versions

Break strategic milestones into implementable versions:

**Guidelines:**
- Each version = 2-4 themes
- Each theme = 2-5 features
- Features should complete in 15-30 minutes
- Group by architectural cohesion, not arbitrary milestone boundaries

**Anti-patterns:**
- One version per roadmap milestone (too coarse)
- One version per feature (too fine)
- Ignoring dependencies between milestones

### 4. Record Scoping Decisions

Document why boundaries were drawn:
- Why these milestones grouped together?
- What was explicitly deferred?
- What dependencies drove sequencing?

This context prevents re-litigating decisions and helps future maintainers.

### 5. Update After Each Version

Post-retrospective updates:
- Mark version complete with date
- Link to retrospective
- Reference key learnings (LRN-XXX)
- Note any backlog items created
- Adjust future version scopes if needed

### 6. Handle Scope Changes

When strategic priorities shift:
- Update affected version mappings
- Move displaced work to backlog with appropriate tags
- Document the change rationale

## Document Template

```markdown
# {Project Name} - Development Plan

> Bridge between strategic roadmap and auto-dev execution.
> 
> Strategic Roadmap: `{path/to/roadmap.md}`
> Last Updated: {date}

## Roadmap → Version Mapping

| Version | Roadmap Reference | Focus | Prerequisites | Status |
|---------|-------------------|-------|---------------|--------|
| v001 | Phase 1, M1.1-1.3 | Foundation + core setup | EXP-001 | planned |
| v002 | Phase 1, M1.4-1.5 | Database + integration | v001 | planned |
| v003 | Phase 1, M1.6-1.7 | API layer | v002 | planned |

## Investigation Dependencies

Track explorations that must complete before version design.

| ID | Question | Informs | Status | Results |
|----|----------|---------|--------|---------|
| EXP-001 | How does X work? | v001 | complete | [link] |
| EXP-002 | Which pattern for Y? | v002 | pending | - |

## Scoping Decisions

### v001 Boundary

**Included:** Milestones 1.1, 1.2, 1.3
**Rationale:** These form a cohesive foundation unit — project setup and core library integration must happen together.
**Deferred:** Database (M1.4) deferred to v002 because it depends on core patterns established in v001.

### v002 Boundary

**Included:** Milestones 1.4, 1.5
**Rationale:** Database and external integration share similar patterns and testing approaches.

## Completed Versions

### v001 - Foundation
- **Completed:** {date}
- **Retrospective:** [VERSION_SUMMARY.md](../versions/v001/VERSION_SUMMARY.md)
- **Key Learnings:** LRN-001, LRN-002
- **Backlog Created:** BL-003 (discovered edge case), BL-004 (deferred enhancement)
- **Notes:** Took longer than expected due to {reason}. Adjusted v002 scope accordingly.

## Backlog Integration

Work that surfaces during planning or execution but doesn't fit current scope:

| Tag | Purpose |
|-----|---------|
| `investigation` | Needs exploration before implementation |
| `deferred` | Known work explicitly pushed to later |
| `discovered` | Found during execution, not originally planned |
| `blocked` | Waiting on external dependency |

Query backlog: `list_backlog_items(project="...", tags=["investigation"])`

## Change Log

| Date | Change | Rationale |
|------|--------|-----------|
| {date} | Initial plan created | Project bootstrap |
| {date} | v003 scope expanded | EXP-002 revealed additional work |
```

## Outputs

- `docs/auto-dev/PLAN.md` created or updated
- Investigation dependencies identified
- Version boundaries defined with rationale

## MCP Tools

| Tool | Usage |
|------|-------|
| `start_exploration` | Run investigations for unknowns |
| `get_exploration_result` | Retrieve investigation findings |
| `add_backlog_item` | Park discovered work |
| `list_backlog_items` | Query backlog by tags |
| `search_learnings` | Find relevant past learnings |
| `list_versions` | See completed versions |

## Next Step

[01-VERSION-SCOPING.md](./01-VERSION-SCOPING.md) - Scope a specific version from the plan
