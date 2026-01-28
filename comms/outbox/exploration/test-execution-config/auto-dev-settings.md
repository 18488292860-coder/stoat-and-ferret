# Auto-Dev MCP Test-Related Settings

## How Auto-Dev MCP Works

The auto-dev-mcp system orchestrates feature implementation by:

1. **Creating prompts** in `comms/inbox/versions/execution/` folders
2. **Invoking Claude Code** with those prompts
3. **Claude Code reads instructions** from:
   - `AGENTS.md` - Project-level guidance
   - `docs/auto-dev/PROMPTS/feature-implementation.md` - Implementation template
   - Feature-specific `requirements.md` and `implementation-plan.md`

## Where Test Commands Are Specified

### 1. AGENTS.md (Primary)

Claude Code reads this file first. It specifies:

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80
```

### 2. Feature Implementation Template

`docs/auto-dev/PROMPTS/feature-implementation.md`:

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest -v
```

### 3. Version Starter Prompts

`comms/inbox/versions/execution/v002/STARTER_PROMPT.md`:

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest -v
```

## No Dynamic Configuration

Currently there is **no mechanism** to:

- Configure test commands per-environment
- Detect WDAC and switch to alternative commands
- Pass custom test runners through MCP configuration

The auto-dev-mcp server spawns Claude Code with prompts, but the test execution strategy is entirely determined by the markdown documentation Claude reads.

## Quality Gate Reporting

Completion reports include quality gate status:

```yaml
quality_gates:
  ruff: pass
  mypy: pass
  pytest: pass
  cargo_clippy: pass
  cargo_test: pass
```

When tests can't run due to WDAC, they're marked as `pending_ci`:

```yaml
quality_gates:
  ruff: pending_ci
  mypy: pending_ci
  pytest: pending_ci
  cargo_clippy: pass
  cargo_test: pass
```

## Auto-Dev Process Flow

```
auto-dev-mcp
    │
    ├── Creates prompt files in comms/inbox/
    │
    ├── Invokes Claude Code
    │       │
    │       ├── Reads AGENTS.md
    │       ├── Reads implementation plan
    │       ├── Implements feature
    │       ├── Runs quality gates (uv run pytest, etc.)
    │       └── Creates completion report
    │
    └── Reads completion report from comms/outbox/
```

## Key Insight

**The MCP server doesn't run tests itself** - it delegates entirely to Claude Code, which follows the instructions in the project documentation. Therefore, fixing the WDAC issue requires modifying the project documentation (`AGENTS.md`) rather than MCP server configuration.
