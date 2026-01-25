# AGENTS.md - stoat-and-ferret

AI-driven video editor with hybrid Python/Rust architecture.

## Project Structure

```
stoat-and-ferret/
├── src/                    # Python source (FastAPI, orchestration)
├── rust/stoat_ferret_core/ # Rust crate (filters, timeline math, FFmpeg)
├── stubs/                  # Python type stubs for Rust bindings
├── tests/                  # Python tests (pytest)
├── docs/design/            # Architecture and design documents
├── docs/auto-dev/          # Auto-dev process documentation
└── comms/                  # MCP communication folders
```

## Quality Gates

### Python (ruff, mypy, pytest)

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
uv run pytest tests/ --cov=src --cov-fail-under=80
```

### Rust (clippy, cargo test)

```bash
cd rust/stoat_ferret_core
cargo clippy -- -D warnings
cargo test
cargo llvm-cov --fail-under-lines 90
```

### Coverage Thresholds

- Python: 80% minimum
- Rust: 90% minimum

## Coding Standards

### Python

- Type hints required on all public functions
- Docstrings required on all public modules, classes, and functions
- Follow existing patterns in codebase
- Use `from __future__ import annotations` for forward references

### Rust

- All public items must have doc comments
- No `unwrap()` in library code - use proper error handling
- Prefer returning `Result<T, E>` over panicking
- PyO3 bindings should have Python-friendly error messages

## PR Workflow

1. Create feature branch from `main`
2. All quality gates must pass before merge
3. Squash merge to maintain clean history
4. Delete branch after merge

## Key Architecture Decisions

- **Non-destructive editing**: Never modify source files
- **Rust for compute**: Filter generation, timeline math, input sanitization in Rust
- **Python for orchestration**: HTTP layer, API routing, AI discoverability
- **PyO3 bindings**: `from stoat_ferret_core import StoatFerretCore`
- **Transparency**: API responses include generated FFmpeg filter strings

## When Called by auto-dev-mcp

Follow instructions in `comms/inbox/` for version/theme/feature execution.
Write completion reports to `comms/outbox/`.
Reference `docs/auto-dev/PROCESS/` for workflow guidance.

## Design Documents

Authoritative design specs in `docs/design/`:

- 01-roadmap.md - Implementation phases
- 02-architecture.md - System architecture
- 03-prototype-design.md - MVP specification
- 04-technical-stack.md - Technology choices
- 05-api-specification.md - REST API design
- 06-risk-assessment.md - Risk analysis
- 07-quality-architecture.md - Quality attributes
- 08-gui-architecture.md - Web GUI design
