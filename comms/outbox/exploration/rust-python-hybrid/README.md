# PyO3/Maturin Hybrid Python+Rust Workflow

The recommended workflow for hybrid Python+Rust projects uses **maturin** as the build tool with **PyO3** bindings. During development, use `maturin develop` for fast iteration—edit Rust code, run `maturin develop`, then test in Python. For distribution, use `maturin build --release` to create wheels, or leverage `maturin-action` in GitHub Actions for cross-platform CI/CD with automatic PyPI publishing.

## Contents

| Document | Description |
|----------|-------------|
| [dev-workflow.md](./dev-workflow.md) | Step-by-step developer workflow: project setup, edit-compile-test cycle, debugging |
| [stub-generation.md](./stub-generation.md) | How to generate and maintain `.pyi` type stubs for PyO3 bindings |
| [ci-setup.md](./ci-setup.md) | GitHub Actions configuration for hybrid Python+Rust projects |
| [gotchas.md](./gotchas.md) | Common pitfalls and how to avoid them |

## Key Tools

- **[maturin](https://www.maturin.rs/)** - Build and publish PyO3/Rust Python packages
- **[PyO3](https://pyo3.rs/)** - Rust bindings for Python
- **[maturin-action](https://github.com/PyO3/maturin-action)** - GitHub Action for CI/CD
- **[pyo3-stub-gen](https://github.com/Jij-Inc/pyo3-stub-gen)** - Type stub generator

## Notable Projects Using This Stack

- [pydantic-core](https://github.com/pydantic/pydantic-core) - Pydantic's Rust core
- [polars](https://github.com/pola-rs/polars) - Fast DataFrame library
- [ruff](https://github.com/astral-sh/ruff) - Python linter
- [orjson](https://github.com/ijl/orjson) - Fast JSON library
