# PyO3 Method Chaining with PyRefMut

## Pattern

When implementing fluent builder APIs in Rust that need to work identically in Python, use `PyRefMut<'_, Self>` as the return type for methods that modify self.

## Example

```rust
#[pymethods]
impl FFmpegCommand {
    fn input(mut slf: PyRefMut<'_, Self>, path: &str) -> PyRefMut<'_, Self> {
        slf.inputs.push(path.to_string());
        slf
    }
    
    fn output(mut slf: PyRefMut<'_, Self>, path: &str) -> PyRefMut<'_, Self> {
        slf.output = Some(path.to_string());
        slf
    }
}
```

## Usage

Works identically in both languages:

**Rust:**
```rust
let cmd = FFmpegCommand::new()
    .input("a.mp4")
    .output("b.mp4")
    .build();
```

**Python:**
```python
cmd = FFmpegCommand().input("a.mp4").output("b.mp4").build()
```

## Why This Works

- `PyRefMut` is PyO3's mutable reference wrapper
- Returning `PyRefMut<'_, Self>` allows chaining without cloning
- The lifetime `'_` ties the reference to the Python GIL
- Method chaining feels natural in both languages

## When to Use

- Builder patterns
- Fluent configuration APIs
- Any method that modifies self and should return self for chaining

## Source

Discovered during stoat-and-ferret v001, Theme 03 (command-builder).
