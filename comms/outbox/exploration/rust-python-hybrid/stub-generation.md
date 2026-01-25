# Type Stub Generation for PyO3 Bindings

## Overview

Type stubs (`.pyi` files) provide type hints for Python tools like mypy, pyright, and IDEs. For PyO3 projects, there are two approaches:

1. **pyo3-stub-gen** (recommended) - Third-party crate with mature support
2. **PyO3 experimental-inspect** - Built-in but still experimental

## Option 1: pyo3-stub-gen (Recommended)

The [pyo3-stub-gen](https://github.com/Jij-Inc/pyo3-stub-gen) crate provides semi-automatic stub generation with manual override capability.

### Setup

Add to `Cargo.toml`:

```toml
[dependencies]
pyo3 = { version = "0.23", features = ["abi3-py310"] }
pyo3-stub-gen = "0.17"

[lib]
crate-type = ["cdylib", "rlib"]  # rlib needed for stub generator binary
```

### Annotate Your Code

```rust
use pyo3::prelude::*;
use pyo3_stub_gen::{derive::gen_stub_pyfunction, define_stub_info_gatherer};

#[gen_stub_pyfunction]  // Add BEFORE #[pyfunction]
#[pyfunction]
fn add(a: i64, b: i64) -> i64 {
    a + b
}

#[gen_stub_pyfunction]
#[pyfunction]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[pymodule]
fn my_module(m: &Bound<PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(add, m)?)?;
    m.add_function(wrap_pyfunction!(greet, m)?)?;
    Ok(())
}

// Required: defines the stub info gatherer
define_stub_info_gatherer!(stub_info);
```

### Create Stub Generator Binary

Create `src/bin/stub_gen.rs`:

```rust
use pyo3_stub_gen::Result;

fn main() -> Result<()> {
    let stub = my_module::stub_info()?;
    stub.generate()?;
    Ok(())
}
```

### Generate Stubs

```bash
cargo run --bin stub_gen
```

This creates `my_module.pyi` which maturin automatically includes in wheels.

### Type Override Examples

When automatic translation isn't sufficient:

```rust
// Complete override with Python syntax
#[gen_stub_pyfunction(python = r#"
def process_items(items: list[str]) -> dict[str, int]: ...
"#)]
#[pyfunction]
fn process_items(items: Vec<String>) -> HashMap<String, i64> {
    // ...
}

// Partial override for specific argument
#[gen_stub_pyfunction]
#[pyfunction]
fn with_callback(
    #[gen_stub(override_type(
        type_repr = "collections.abc.Callable[[str], None]",
        imports = ("collections.abc",)
    ))]
    callback: Bound<PyAny>,
) -> PyResult<()> {
    // ...
}
```

### Skip Functions

```rust
#[gen_stub(skip)]
fn internal_helper(&self) {
    // Not included in .pyi
}
```

## Option 2: PyO3 Experimental Inspect

PyO3 has built-in experimental support (still in development).

### Enable Feature

```toml
[dependencies]
pyo3 = { version = "0.23", features = ["experimental-inspect"] }
```

### Add Type Hints in Signatures

```rust
#[pyfunction]
#[pyo3(signature = (items: "list[int]") -> "list[int]")]
fn double_all(items: Vec<i64>) -> Vec<i64> {
    items.into_iter().map(|x| x * 2).collect()
}
```

### Limitations

- Only inline modules supported
- Custom types need manual `FromPyObject::INPUT_TYPE` implementation
- Tool integration (maturin) not yet complete
- Many features unimplemented

## Manual Stubs (Fallback)

For full control, maintain stubs manually:

```
my_project/
├── python/
│   └── my_project/
│       ├── __init__.py
│       └── _core.pyi  # Manual stub file
└── src/
    └── lib.rs
```

Example `_core.pyi`:

```python
def add(a: int, b: int) -> int: ...
def greet(name: str) -> str: ...

class MyClass:
    def __init__(self, value: int) -> None: ...
    def get_value(self) -> int: ...
```

## Best Practices

1. **Use pyo3-stub-gen** for new projects - it's the most mature solution
2. **Keep stubs in sync** - regenerate after API changes
3. **Test with mypy/pyright** - verify stubs work correctly
4. **Document complex types** - add docstrings for non-obvious conversions

## References

- [pyo3-stub-gen Documentation](https://docs.rs/pyo3-stub-gen)
- [PyO3 Type Stub Generation](https://pyo3.rs/main/type-stub.html)
- [PyO3 Typing Hints Appendix](https://pyo3.rs/main/python-typing-hints.html)
