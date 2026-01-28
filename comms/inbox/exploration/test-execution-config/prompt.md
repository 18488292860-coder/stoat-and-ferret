## Auto-Dev MCP Test Execution Configuration

Investigate how auto-dev-mcp currently handles test execution and whether configuration changes could help with the WDAC issue.

### Context
- The auto-dev-mcp system uses Claude Code to execute features
- Claude Code needs to run pytest for Python projects
- Windows WDAC may be blocking .venv executables
- We need to understand what configuration options exist

### Research Tasks

1. **Check auto-dev-mcp test execution configuration**
   - Read `docs/auto-dev/PROCESS/` for any test configuration docs
   - Check if there's a way to customize test commands
   - Look for any pytest-related configuration in the auto-dev setup

2. **Examine CLAUDE.md for test instructions**
   - Read the project's CLAUDE.md file
   - Check what test commands are specified
   - See if there are any Windows-specific instructions

3. **Check pyproject.toml test configuration**
   - Review [tool.pytest.ini_options] section
   - Check for any custom test paths or settings
   - Look for any scripts defined that run tests

4. **Research Claude Code execution environment**
   - How does Claude Code execute bash/PowerShell commands?
   - Does it have access to system PATH?
   - Can it use different Python interpreters?

5. **Check if there are existing workarounds in the project**
   - Search for any scripts that handle test execution
   - Look for any CI configuration that might have solutions
   - Check .github/workflows/ for test execution patterns

6. **Examine the completion report generation process**
   - How does auto-dev-mcp run quality gates?
   - Can we customize the pytest command used?
   - Is there a fallback when tests can't run?

### Output Requirements

Save all findings to: `comms/outbox/exploration/test-execution-config/`

Create these files:
- `README.md` - Summary of test execution flow
- `current-config.md` - Current test configuration in project
- `auto-dev-settings.md` - Auto-dev MCP test-related settings
- `modification-options.md` - What can be changed to fix the issue

Commit with message: "docs(exploration): test execution configuration analysis"
