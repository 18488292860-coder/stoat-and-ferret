# Current Test Configuration

## pyproject.toml Settings

The project's pytest configuration in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing"
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
]

[tool.coverage.run]
source = ["src"]
branch = true

[tool.coverage.report]
fail_under = 80
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
]
```

## Specified Quality Gate Commands

From `AGENTS.md`:

```bash
# Python (ruff, mypy, pytest)
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80

# Rust (clippy, cargo test)
cd rust/stoat_ferret_core
cargo clippy -- -D warnings
cargo test
```

## Coverage Requirements

- **Python**: 80% minimum coverage
- **Rust**: 90% minimum coverage (documented, not enforced by config)

## CI Workflow Configuration

From `.github/workflows/ci.yml`:

```yaml
- name: Python tests
  run: uv run pytest tests/ -v --cov=src --cov-fail-under=80
```

The CI uses the same `uv run pytest` pattern as local development.

## Test Discovery

- Tests are in `tests/` directory
- Pytest discovers tests automatically via standard naming conventions
- Current test file: `tests/test_pyo3_bindings.py`

## Dependencies

From `pyproject.toml` dev dependencies:

```toml
[dependency-groups]
dev = [
    "pytest>=8.0",
    "pytest-cov>=4.0",
    "ruff>=0.4",
    "mypy>=1.10",
    "maturin>=1.0",
]
```

## Key Observations

1. **No scripts defined** - There are no npm-style scripts or Makefile targets for running tests
2. **Direct tool invocation** - Quality gates call tools directly via `uv run`
3. **No path customization** - Tests and source paths are standard, no special handling
4. **Rust tests separate** - Rust tests use cargo directly, not affected by WDAC
