# Modification Options for WDAC Issue

## Problem Summary

Windows Defender Application Control (WDAC) blocks `.venv/Scripts/*.exe` execution, preventing:
- `uv run pytest`
- `uv run ruff`
- `uv run mypy`

Rust tests (`cargo test`, `cargo clippy`) work fine because they don't use the venv.

## Option 1: Alternative Test Commands in AGENTS.md

**Recommendation: Best short-term solution**

Add fallback instructions to `AGENTS.md` for WDAC-affected environments:

```markdown
## Quality Gates (WDAC Workaround)

If `uv run` commands fail with "blocked by policy" errors, try these alternatives:

### Using uvx (uv tool execution)
uvx pytest tests/ --cov=src --cov-fail-under=80
uvx ruff check src/ tests/
uvx ruff format --check src/ tests/
uvx mypy src/

### Using system Python (if installed globally)
python -m pytest tests/ --cov=src --cov-fail-under=80
python -m ruff check src/ tests/
python -m mypy src/
```

**Pros:**
- No code changes required
- Claude Code can read and follow instructions
- Easy to implement

**Cons:**
- Relies on Claude Code recognizing and adapting to WDAC errors
- May need explicit detection/fallback logic

## Option 2: Create Test Runner Script

Create a `scripts/run-tests.py` or `scripts/run-tests.ps1` that:
1. Tries `uv run pytest`
2. Falls back to `uvx pytest`
3. Falls back to `python -m pytest`

```python
#!/usr/bin/env python
import subprocess
import sys

def run_tests():
    commands = [
        ["uv", "run", "pytest", "tests/", "-v"],
        ["uvx", "pytest", "tests/", "-v"],
        [sys.executable, "-m", "pytest", "tests/", "-v"],
    ]

    for cmd in commands:
        try:
            result = subprocess.run(cmd, check=True)
            return result.returncode
        except (subprocess.CalledProcessError, FileNotFoundError, PermissionError):
            continue

    print("All test execution methods failed")
    return 1

if __name__ == "__main__":
    sys.exit(run_tests())
```

**Pros:**
- Automatic fallback without Claude Code interpretation
- Consistent behavior

**Cons:**
- Requires the script itself to be executable (may hit same WDAC issue)
- Additional maintenance

## Option 3: Use uvx Instead of uv run

Change all `uv run` commands in AGENTS.md to `uvx`:

```markdown
## Commands
uvx ruff check .          # Lint
uvx ruff format .         # Format
uvx mypy src/             # Type check
uvx pytest                # Test
```

**How uvx differs:** `uvx` runs tools in isolated environments without depending on `.venv/Scripts/`.

**Pros:**
- Single-line changes
- May bypass WDAC if uvx uses different paths

**Cons:**
- Needs testing to confirm uvx isn't also blocked
- May have different behavior than venv-based execution

## Option 4: Conditional Instructions Based on Error Detection

Update `AGENTS.md` with explicit WDAC detection:

```markdown
## Quality Gates

Run these commands. If you see "blocked by policy" or "Access is denied" errors:
1. Note in completion-report.md: `pytest: pending_ci`
2. Ensure Rust tests pass
3. CI will verify Python tests

### Primary Commands
uv run pytest tests/ --cov=src --cov-fail-under=80

### If Blocked by WDAC
- Mark Python quality gates as `pending_ci`
- Continue with PR workflow
- CI will catch any issues
```

**Pros:**
- Acknowledges reality of enterprise Windows environments
- Doesn't block development progress

**Cons:**
- Defers Python testing to CI
- May miss issues caught locally

## Option 5: WSL or Docker Container

Document WSL/Docker as the development environment for WDAC-restricted machines:

```markdown
## Windows with WDAC

If Windows Application Control blocks test execution:

### Option A: WSL
wsl -e bash -c "cd /mnt/c/path/to/project && uv run pytest"

### Option B: Docker
docker run -v .:/app -w /app python:3.11 bash -c "pip install uv && uv sync && uv run pytest"
```

**Pros:**
- Complete bypass of WDAC
- Full Linux-like environment

**Cons:**
- Requires WSL/Docker setup
- More complex for Claude Code to execute
- Performance overhead

## Recommended Approach

**Immediate (Option 4):** Update AGENTS.md to acknowledge WDAC limitations and allow `pending_ci` status for Python quality gates when blocked.

**Short-term (Option 3):** Test if `uvx` works around WDAC. If yes, switch to uvx commands.

**Medium-term (Option 1):** Add fallback command documentation with explicit alternatives.

## Implementation

To implement Option 4, add to `AGENTS.md`:

```markdown
### WDAC Environments (Windows)

On Windows machines with Application Control policies, `.venv` executables may be blocked. When you encounter "blocked by policy" errors:

1. **Rust tests still work** - `cargo clippy` and `cargo test` are unaffected
2. **Mark Python gates as pending_ci** in completion-report.md:
   ```yaml
   quality_gates:
     ruff: pending_ci
     mypy: pending_ci
     pytest: pending_ci
     cargo_clippy: pass
     cargo_test: pass
   ```
3. **Continue with PR workflow** - CI will verify Python quality gates
4. **Document in quality-gaps.md** if this affects specific acceptance criteria
```
