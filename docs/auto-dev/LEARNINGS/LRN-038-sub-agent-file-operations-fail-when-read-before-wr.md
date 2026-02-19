## Context

When using AI sub-agents (Task tool spawns) to perform file operations as part of automated workflows, including documentation generation and code analysis.

## Insight

Sub-agents frequently attempt to Write or Edit files they haven't Read first, causing repeated "File has not been read yet" errors. This wastes API round-trips and can cascade into sibling tool call failures when parallel operations are used. Sub-agent prompts should explicitly instruct reading target files before any write/edit operations, and the Read step should be a prerequisite in any file modification workflow.

## Evidence

Session analytics showed recurring patterns across multiple sub-agent sessions: `agent-aaf973a` had 3 Write errors, `agent-a2d8a40` had 8+ Edit errors, `agent-ad2d48f` had 3 Write errors, `agent-af73f3f` had 2 Write errors — all "File has not been read yet" failures. These appeared across C4 documentation generation, exploration tasks, and analysis sub-agents. The pattern represents wasted API round-trips where sub-agents attempted to create/modify output files without the required prior Read call.

## Application

- Include explicit "Read before Write/Edit" instructions in sub-agent prompts
- Structure sub-agent workflows as: Read target file -> process -> Write/Edit result
- When creating new output files, note that Write still requires a prior Read attempt
- Monitor sub-agent error rates for this pattern as a quality signal for prompt effectiveness