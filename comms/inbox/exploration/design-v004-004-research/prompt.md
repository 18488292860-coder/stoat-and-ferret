You are performing Task 004: Research and Investigation for stoat-and-ferret version v004.

Read AGENTS.md first and follow all instructions there.

PROJECT=stoat-and-ferret
VERSION=v004

## Objective

Research unknowns, investigate prior art, and gather evidence for design decisions that affect version STRUCTURE (not implementation details Claude Code can figure out).

## Context

Tasks 001-003 gathered context. This task resolves unknowns before proposing solutions.

Read design artifact store:
- `comms/outbox/versions/design/v004/001-environment/README.md` — environment context
- `comms/outbox/versions/design/v004/002-backlog/backlog-details.md` — full backlog details
- `comms/outbox/versions/design/v004/002-backlog/retrospective-insights.md` — retrospective insights
- `comms/outbox/versions/design/v004/003-impact-assessment/impact-table.md` — impacts identified

## Tasks

### 1. Identify Research Questions

Based on backlog items from Task 002, identify:
- Technologies or libraries requiring investigation
- Unclear areas of existing codebase
- Patterns needed but not yet understood
- Integration points with existing systems

Create a list of specific research questions.

### 2. Codebase Investigation

For questions about existing code:
- Use `request_clarification` to search for patterns, classes, or files
- Document findings with file paths and line references

Key areas to investigate:
- Current test infrastructure (pytest configuration, conftest.py, fixtures)
- Current dependency injection patterns in create_app() 
- InMemory repository implementations
- FFmpeg executor patterns and test doubles
- Search behavior differences (InMemory vs FTS5)
- Current security patterns for Rust sanitization
- Async patterns in the codebase

### 3. Verify Referenced Learnings

Check any learnings referenced in backlog items or retrospective. For each:
1. Read the learning
2. Verify the condition still exists in codebase
3. Mark as STALE or VERIFIED

### 4. External Research

Use DeepWiki for relevant library questions:
- pytest fixtures best practices for FastAPI apps
- Black box testing patterns for REST APIs
- Property-based testing with Hypothesis
- FFmpeg contract testing approaches
- Async job queue patterns (asyncio)

### 5. Evidence Gathering

For concrete values needed (timeouts, limits, thresholds):
- Query existing codebase for current values
- Check configuration files
- Mark undeterminable items as "TBD - requires runtime testing"

### 6. Impact Analysis

Document:
- Dependencies: What existing code/tools/configs will change
- Breaking changes: Will existing workflows or APIs break
- Test impact: What tests need creation or update
- Documentation: What docs require update after implementation

## Output Requirements

Create findings in comms/outbox/exploration/design-v004-004-research/:

### README.md (required)

First paragraph: Summary of research scope and key findings.

Then:
- **Research Questions**: List of questions investigated
- **Findings Summary**: High-level results
- **Unresolved Questions**: Items that cannot be resolved pre-implementation
- **Recommendations**: Design direction based on research

### codebase-patterns.md

Findings from codebase investigation: existing patterns, file paths, key classes/functions.

### external-research.md

Findings from DeepWiki and web research: libraries/tools evaluated, patterns found, sources cited.

### evidence-log.md

For all concrete values needed with value, source, data, and rationale.

### impact-analysis.md

Analysis of implementation impact: dependencies, breaking changes, test needs, doc updates.

## Guidelines

- ALL backlog items from PLAN.md are MANDATORY — research must cover all of them
- Document sources for all findings
- Mark truly undeterminable items as "TBD - requires [X]"
- Keep each document under 200 lines

## When Complete

After writing all output files to comms/outbox/exploration/design-v004-004-research/, also COPY all output files to comms/outbox/versions/design/v004/004-research/ so they are in the design artifact store.

git add comms/outbox/exploration/design-v004-004-research/
git commit -m "exploration: design-v004-004-research - research investigation complete"
