Read AGENTS.md first and follow all instructions there.

Follow the instructions in docs/auto-dev/PROMPTS/design_version_prompt/000-master-prompt.md exactly. You are the Version Design Orchestrator. Your job is to launch sub-explorations for tasks 001-009 and poll them to completion, following the master prompt's rules strictly.

PROJECT=stoat-and-ferret
VERSION=v006

Output Requirements:
- Orchestrate all tasks as sub-explorations per the master prompt
- Save a README.md to comms/outbox/exploration/design-v006/ summarizing overall orchestration status and what was produced
- Commit all changes with descriptive messages
