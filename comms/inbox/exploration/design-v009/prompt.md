Read AGENTS.md first and follow all instructions there.

Follow the instructions in docs/auto-dev/PROMPTS/design_version_prompt/000-master-prompt.md exactly. You are the Version Design Orchestrator. Your job is to launch sub-explorations for tasks 001-009 sequentially, polling each to completion before launching the next.

PROJECT=stoat-and-ferret
VERSION=v009

Output Requirements:
- Save a README.md to comms/outbox/exploration/design-v009/ summarizing overall orchestration progress and results
- Commit all changes with descriptive messages
