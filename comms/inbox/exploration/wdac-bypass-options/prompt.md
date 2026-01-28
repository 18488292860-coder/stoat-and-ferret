## WDAC Bypass Options Research

Research potential solutions for running pytest when Windows Defender Application Control blocks .venv executables.

### Context
- Windows Application Control (WDAC/AppLocker) is blocking Python executables in .venv
- Need to find ways to run tests that work within security constraints
- Solutions must be practical for automated CI/development workflows

### Research Tasks

1. **Document current Python/uv configuration**
   - Check `which python`, `which uv`, `which pytest`
   - Check `uv --version`, `python --version`
   - Check if there's a system-wide Python installation
   - Run `where python` and `where pytest` to find all installations

2. **Test system Python approach**
   - Can we install pytest globally? `pip install --user pytest`
   - Can we run tests with system Python pointing to project? 
   - Test: `python -m pytest tests/ -v` (if system python exists)

3. **Test uv tool run approach**
   - Try `uv tool run pytest tests/ -v`
   - Try `uvx pytest tests/ -v`
   - These may use a different execution path

4. **Research WDAC exceptions**
   - Can we whitelist specific paths?
   - Is there a way to sign the venv executables?
   - Can we use a different venv location that's allowed?

5. **Check pyproject.toml for test configuration**
   - Review current pytest configuration
   - Check if there are any path-specific settings

6. **Test Docker/WSL alternative**
   - Check if WSL is available: `wsl --list`
   - Check if Docker is available: `docker --version`
   - These would bypass Windows WDAC entirely

### Output Requirements

Save all findings to: `comms/outbox/exploration/wdac-bypass-options/`

Create these files:
- `README.md` - Summary with recommended approach
- `system-python-option.md` - Details on system Python approach
- `uv-tool-option.md` - Details on uv tool run approach
- `containerization-option.md` - WSL/Docker findings
- `recommended-solution.md` - Final recommendation with implementation steps

Commit with message: "docs(exploration): WDAC bypass options for pytest"
