# Common Pitfalls and Best Practices

## Build Configuration Issues

### Module Name Mismatch

**Problem:** Python can't import your module.

**Cause:** The `#[pymodule]` function name doesn't match `lib.name` in `Cargo.toml`.

```rust
// lib.rs
#[pymodule]
fn my_module(m: &Bound<PyModule>) -> PyResult<()> { ... }  // Must match!
```

```toml
# Cargo.toml
[lib]
name = "my_module"  # Must match!
```

### Extension Module Feature (PyO3 < 0.27)

**Problem:** Linker errors about Python symbols.

**Fix:** Enable the extension-module feature:

```toml
# pyproject.toml
[tool.maturin]
features = ["pyo3/extension-module"]
```

Note: PyO3 0.27+ handles this automatically.

### cargo test Linker Errors

**Problem:** `cargo test` fails with symbol errors.

**Cause:** cdylib doesn't support Rust tests well.

**Fix:** Use dual crate types:

```toml
[lib]
crate-type = ["cdylib", "rlib"]
```

Then make extension-module optional:

```toml
# Cargo.toml
[features]
extension-module = ["pyo3/extension-module"]
```

```toml
# pyproject.toml
[tool.maturin]
features = ["extension-module"]
```

## Runtime Issues

### Ctrl+C Not Working

**Problem:** Can't interrupt long-running Rust code with Ctrl+C.

**Cause:** Python's SIGINT handler can't run while Rust executes.

**Fix:** Periodically check for signals:

```rust
#[pyfunction]
fn long_running(py: Python<'_>) -> PyResult<()> {
    for i in 0..1_000_000 {
        if i % 1000 == 0 {
            py.check_signals()?;  // Allow Ctrl+C
        }
        // ... work ...
    }
    Ok(())
}
```

### Deadlocks with lazy_static/OnceLock

**Problem:** Deadlock when using `OnceLock` or `lazy_static!`.

**Cause:** GIL interaction with Rust synchronization primitives.

**Fix:** Use PyO3's `PyOnceLock`:

```rust
use pyo3::sync::PyOnceLock;

static CACHE: PyOnceLock<Vec<String>> = PyOnceLock::new();

fn get_cache(py: Python<'_>) -> &Vec<String> {
    CACHE.get_or_init(py, || vec!["cached".to_string()])
}
```

### Field Returns New Object Each Time

**Problem:** `#[pyo3(get)]` returns new Python objects on each access.

**Cause:** Rust ownership requires cloning.

**Fix:** Use `Py<T>` for heap-allocated Python objects:

```rust
#[pyclass]
struct Container {
    #[pyo3(get)]
    data: Py<PyList>,  // Returns same object each time
}
```

## Distribution Issues

### Manylinux Compatibility

**Problem:** Wheels don't work on older Linux systems.

**Cause:** Built against newer glibc.

**Fix:** Use appropriate manylinux tag:

```yaml
# GitHub Actions
- uses: PyO3/maturin-action@v1
  with:
    manylinux: 2014  # glibc 2.17, works on most systems
```

Rust 1.64+ requires at least glibc 2.17 (manylinux2014).

### Windows DLL Not Found

**Problem:** `STATUS_DLL_NOT_FOUND` on Windows.

**Cause:** Python DLL not in PATH.

**Fix options:**
1. Add Python to PATH
2. Use PyOxidizer for bundled distribution
3. Ship DLL alongside your wheel

### Binary + Library Size

**Problem:** Including both a CLI binary and library doubles wheel size.

**Fix:** Use Python entrypoint instead:

```toml
# pyproject.toml
[project.scripts]
my-cli = "my_module:cli_main"
```

```rust
#[pyfunction]
fn cli_main() -> PyResult<()> {
    // CLI logic here
}
```

## Code Organization

### Module Path Confusion

**Problem:** Tools like mkdocs can't find the correct module path.

**Cause:** Maturin's auto-generated `__init__.py` creates nested paths like `my_module.my_module.MyClass`.

**Fix:** Use mixed Rust/Python layout with explicit `__init__.py`:

```
my_project/
├── python/
│   └── my_project/
│       └── __init__.py  # Control exports here
└── src/
    └── lib.rs
```

```toml
# pyproject.toml
[tool.maturin]
python-source = "python"
module-name = "my_project._core"
```

### pyclass Module Attribute

**Problem:** `__module__` attribute shows wrong path.

**Fix:** Set explicitly:

```rust
#[pyclass(module = "my_project")]
struct MyClass { ... }
```

## Python Version Support

### Minimum Python Version

**Problem:** pyo3-stub-gen requires Python 3.10+.

**Fix:** Don't enable Python 3.9 or older in PyO3 settings when using pyo3-stub-gen.

### ABI3 for Multi-Version Wheels

**Benefit:** One wheel works across Python versions.

```toml
[dependencies]
pyo3 = { version = "0.23", features = ["abi3-py310"] }
```

## Performance

### Debug vs Release Builds

**Problem:** Development builds are slow.

**Fix:** Use release builds for benchmarking:

```bash
maturin develop --release
```

### Unnecessary Clones

**Problem:** Performance regression from excessive cloning.

**Fix:** Use references where possible, leverage PyO3's borrowing:

```rust
#[pyfunction]
fn process(data: &str) -> String {  // Borrows, no clone
    data.to_uppercase()
}
```

## References

- [PyO3 FAQ](https://pyo3.rs/v0.27.2/faq.html)
- [Maturin Troubleshooting](https://www.maturin.rs/tutorial.html)
- [PyO3 Building and Distribution](https://pyo3.rs/v0.27.2/building-and-distribution.html)
