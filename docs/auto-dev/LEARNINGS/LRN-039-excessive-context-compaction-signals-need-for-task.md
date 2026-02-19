## Context

When AI agent sessions are long-running and involve reading many files, processing large codebases, or performing multi-step implementation tasks.

## Insight

High context compaction frequency (3+ events per session, with some sessions hitting 10-16) indicates sessions are consuming context faster than productive work requires. This suggests tasks should be decomposed into smaller sub-tasks, file reading should be more targeted, or intermediate results should be summarized before moving to the next step. When nearly half of sessions exhibit this pattern, it is systemic rather than incidental.

## Evidence

Session health analysis found 24 of ~50 sessions (48%) had 3+ context compaction events. The worst session had 16 compaction events, meaning it exhausted and compressed its context 16 times. Sessions involving C4 documentation generation, multi-feature implementation, and large codebase exploration were most affected. The finding suggests sessions reading many large files without summarizing routinely exceed context limits.

## Application

- Decompose large tasks into focused sub-tasks that each fit within context limits
- Summarize intermediate results before moving to the next phase of work
- Use targeted file reading (specific sections, line ranges) instead of full-file reads when possible
- Monitor compaction count as a health metric: sessions with 5+ compactions likely benefit from decomposition
- Delegate read-heavy research phases to sub-agents to protect the main session context