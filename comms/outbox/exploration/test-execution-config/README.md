# Test Execution Configuration Analysis

## Summary

This exploration investigates how the auto-dev-mcp system handles test execution and what configuration options exist to address Windows WDAC blocking issues.

## Key Findings

### Current State

1. **Test execution is hardcoded in prompts** - The quality gate commands are specified directly in:
   - `AGENTS.md` (project guidance)
   - `docs/auto-dev/PROMPTS/feature-implementation.md`
   - `comms/inbox/versions/execution/*/STARTER_PROMPT.md`

2. **No dynamic configuration** - There is no mechanism to swap out test commands based on environment

3. **Claude Code uses `uv run` pattern** - All Python tool execution uses `uv run <tool>` which invokes tools from `.venv/Scripts/`

4. **CI passes, local execution fails** - The CI workflow works fine because GitHub Actions runners don't have WDAC. The problem is specific to local Windows development environments with Device Guard/WDAC enabled.

### WDAC Impact

Windows Defender Application Control (WDAC) blocks execution of unsigned binaries in certain directories. When `uv sync` creates a virtual environment:
- Python.exe is copied to `.venv/Scripts/python.exe`
- Tool executables (pytest, ruff, mypy) are installed there
- WDAC blocks these unsigned executables

## Files in This Analysis

| File | Description |
|------|-------------|
| [README.md](./README.md) | This summary |
| [current-config.md](./current-config.md) | Current test configuration details |
| [auto-dev-settings.md](./auto-dev-settings.md) | Auto-dev MCP test-related settings |
| [modification-options.md](./modification-options.md) | Options to fix the WDAC issue |

## Quick Recommendation

The most practical short-term solution is to modify `AGENTS.md` to provide alternative test commands that Claude Code can use when WDAC blocks `.venv` execution. See [modification-options.md](./modification-options.md) for details.
