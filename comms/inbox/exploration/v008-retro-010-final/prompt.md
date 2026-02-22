Read AGENTS.md first and follow all instructions there, including the mandatory PR workflow.

Follow the task instructions in docs/auto-dev/PROMPTS/retrospective_prompt/task_prompts/010-finalization.md exactly.

PROJECT=stoat-and-ferret
VERSION=v008

autoDevToolKey: 3bde9d4c-cd4c-4ec4-98e1-b50bdbac0032
Include this autoDevToolKey value in every MCP tool call to auto-dev-mcp.

Note: The task prompt references "commit_changes" but the actual MCP tool for committing is "git_write". Use git_write for all commit operations.

Output Requirements:
- Save task artifacts to comms/outbox/versions/retrospective/v008/010-finalization/ as specified in the task prompt
- Save a README.md to comms/outbox/exploration/v008-retro-010-final/ summarizing what was produced
- Commit all changes with descriptive messages