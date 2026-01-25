# Developer Workflow for PyO3/Maturin Projects

## Project Setup

### 1. Initialize Project

```bash
# Create new project with maturin
pip install maturin
maturin init -b pyo3 my_project
cd my_project
```

This creates the basic structure:

```
my_project/
├── Cargo.toml
├── pyproject.toml
└── src/
    └── lib.rs
```

### 2. Configure Cargo.toml

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[lib]
name = "my_project"
crate-type = ["cdylib", "rlib"]  # cdylib for Python, rlib for tests

[dependencies]
pyo3 = { version = "0.23", features = ["abi3-py310"] }
```

### 3. Configure pyproject.toml

```toml
[build-system]
requires = ["maturin>=1.0,<2.0"]
build-backend = "maturin"

[project]
name = "my_project"
requires-python = ">=3.10"

[tool.maturin]
features = ["pyo3/extension-module"]
```

### 4. Set Up Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# or: .venv\Scripts\activate  # Windows
pip install -U pip maturin pytest
```

## Edit-Compile-Test Cycle

### Development Build

```bash
# After editing Rust code:
maturin develop

# For release-optimized development build:
maturin develop --release
```

`maturin develop` compiles and installs directly into your virtualenv. This is the fastest iteration path.

### Running Tests

```bash
# Python tests
pytest tests/

# Rust unit tests (requires dual crate-type)
cargo test
```

### Example Workflow

```bash
# 1. Edit src/lib.rs
vim src/lib.rs

# 2. Rebuild (fast, debug build)
maturin develop

# 3. Test in Python
python -c "import my_project; print(my_project.my_function())"

# 4. Run test suite
pytest
```

## Mixed Rust/Python Project

For projects with both Rust extensions and Python code:

```
my_project/
├── Cargo.toml
├── pyproject.toml
├── python/
│   └── my_project/
│       ├── __init__.py
│       └── helpers.py
└── src/
    └── lib.rs
```

Configure in `pyproject.toml`:

```toml
[tool.maturin]
python-source = "python"
module-name = "my_project._core"
```

Then in `python/my_project/__init__.py`:

```python
from my_project._core import rust_function
from my_project.helpers import python_helper
```

## Debugging Tips

### Print Debugging in Rust

```rust
#[pyfunction]
fn my_function() -> PyResult<String> {
    eprintln!("Debug: entering my_function");  // Prints to stderr
    Ok("result".to_string())
}
```

### Enable Rust Backtraces

```bash
RUST_BACKTRACE=1 python -c "import my_project; my_project.crash()"
```

### IDE Setup (VS Code)

Install extensions:
- rust-analyzer (Rust)
- Python
- Even Better TOML

For debugging Rust from Python, use `codelldb` extension with a launch configuration.

## Build Commands Reference

| Command | Purpose |
|---------|---------|
| `maturin develop` | Fast dev build, installs to venv |
| `maturin develop -r` | Release-optimized dev build |
| `maturin build` | Build wheel to `target/wheels/` |
| `maturin build --release` | Release wheel for distribution |
| `cargo test` | Run Rust unit tests |
| `cargo check` | Fast syntax/type check |
| `cargo clippy` | Linting |

## References

- [Maturin Tutorial](https://www.maturin.rs/tutorial.html)
- [PyO3 User Guide](https://pyo3.rs/)
- [PyO3 Building and Distribution](https://pyo3.rs/v0.27.2/building-and-distribution.html)
