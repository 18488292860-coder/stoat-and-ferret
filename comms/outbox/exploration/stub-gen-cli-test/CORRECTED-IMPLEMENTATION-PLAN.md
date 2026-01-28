# Corrected Implementation Plan: 001-stub-regeneration

## Problem Summary

The original implementation plan assumed:
1. `stub_gen` accepts `--output-dir` CLI argument (FALSE - no CLI args supported)
2. `stub_gen` runs successfully (FALSE - fails with "file not found")
3. Stubs are auto-generated (FALSE - manually written)

Root cause: `define_stub_info_gatherer!(stub_info)` macro hardcodes `CARGO_MANIFEST_DIR` (`rust/stoat_ferret_core/`) but `pyproject.toml` exists only at project root.

---

## Corrected Steps

### Step 1: Fix pyproject.toml Path in lib.rs

**File:** `rust/stoat_ferret_core/src/lib.rs`

**Action:** Replace the macro with a custom function that navigates to project root.

**Find this line (near end of file):**
```rust
define_stub_info_gatherer!(stub_info);
```

**Replace with:**
```rust
/// Gather stub information from the project root pyproject.toml.
///
/// This replaces the `define_stub_info_gatherer!` macro because the macro
/// looks for pyproject.toml at CARGO_MANIFEST_DIR (rust/stoat_ferret_core/),
/// but our pyproject.toml is at the project root.
pub fn stub_info() -> pyo3_stub_gen::Result<pyo3_stub_gen::StubInfo> {
    let manifest_dir = std::path::Path::new(env!("CARGO_MANIFEST_DIR"));
    // Navigate: rust/stoat_ferret_core/ -> rust/ -> project_root/
    let project_root = manifest_dir
        .parent()
        .expect("Failed to get rust/ directory")
        .parent()
        .expect("Failed to get project root");
    pyo3_stub_gen::StubInfo::from_pyproject_toml(project_root.join("pyproject.toml"))
}
```

**Also update imports at top of file - find:**
```rust
use pyo3_stub_gen::{define_stub_info_gatherer, derive::gen_stub_pyfunction};
```

**Replace with:**
```rust
use pyo3_stub_gen::derive::gen_stub_pyfunction;
```

---

### Step 2: Update pyproject.toml for Stub Output Location

The current config outputs stubs to `src/stoat_ferret_core/` but existing stubs are in `stubs/stoat_ferret_core/`.

**Option A (Recommended):** Keep stubs in `src/` alongside the compiled extension.

No pyproject.toml changes needed. After generation, stubs will appear at:
- `src/stoat_ferret_core/__init__.pyi` (or similar based on module structure)

**Option B:** If stubs MUST stay in `stubs/`, add a post-generation copy step.

For now, proceed with Option A (no pyproject.toml changes).

---

### Step 3: Build and Run stub_gen

```bash
cd rust/stoat_ferret_core
cargo build --bin stub_gen
cargo run --bin stub_gen
```

**Expected output:** No errors. Stub files created in `src/stoat_ferret_core/`.

**Verify:**
```bash
# From project root
ls -la src/stoat_ferret_core/*.pyi
```

---

### Step 4: Compare Generated vs Manual Stubs

```bash
# Compare content (ignore formatting differences)
diff -u stubs/stoat_ferret_core/_core.pyi src/stoat_ferret_core/_core.pyi || true
diff -u stubs/stoat_ferret_core/__init__.pyi src/stoat_ferret_core/__init__.pyi || true
```

Document any significant differences. Generated stubs may have:
- Different formatting
- Missing docstrings (if not annotated in Rust)
- Different type representations

---

### Step 5: Update mypy Configuration

**File:** `pyproject.toml`

If stubs are now in `src/stoat_ferret_core/`, mypy will find them automatically (since `src` is already in the path). 

**Optionally** remove the redundant `stubs` path once generated stubs are verified:

**Find:**
```toml
[tool.mypy]
mypy_path = ["src", "stubs"]
```

**Replace with:**
```toml
[tool.mypy]
mypy_path = ["src"]
```

Then delete or archive the manual stubs:
```bash
rm -rf stubs/stoat_ferret_core/
# OR
mv stubs/stoat_ferret_core/ stubs/stoat_ferret_core.manual-backup/
```

---

### Step 6: Add CI Stub Freshness Check

**File:** `.github/workflows/ci.yml` (or equivalent)

Add a job/step:
```yaml
- name: Verify stubs are up-to-date
  run: |
    cd rust/stoat_ferret_core
    cargo run --bin stub_gen
    git diff --exit-code src/stoat_ferret_core/*.pyi || {
      echo "ERROR: Stubs are out of date. Run 'cargo run --bin stub_gen' and commit the changes."
      exit 1
    }
```

---

### Step 7: Update Test Assertions

Review tests that import from `stoat_ferret_core` and verify they still pass:
```bash
pytest tests/ -v
mypy src/stoat_ferret tests/
```

If tests fail due to type signature changes, update the test assertions to match the generated stubs.

---

### Step 8: Commit Changes

```bash
git add rust/stoat_ferret_core/src/lib.rs
git add src/stoat_ferret_core/*.pyi
git add pyproject.toml  # if modified
git add .github/workflows/ci.yml  # if added CI check
git commit -m "feat(bindings): enable pyo3-stub-gen automatic stub generation

- Fix stub_info() to use project root pyproject.toml
- Generate stubs to src/stoat_ferret_core/
- Add CI verification for stub freshness"
```

---

## Key Differences from Original Plan

| Original | Corrected |
|----------|-----------|
| Assumed `--output-dir` flag exists | No CLI args - output controlled by pyproject.toml |
| "Audit current stub binary" (investigative) | "Replace macro with custom function" (prescriptive) |
| "Identify test breakages" (~20 unknown) | "Run pytest and mypy, fix failures" (concrete) |
| Conditional: "modify if needed" | Exact code changes specified |
| Unknown output location | Output to `src/stoat_ferret_core/` per config |

---

## Verification Checklist

- [ ] `cargo run --bin stub_gen` exits with code 0
- [ ] `.pyi` files created in `src/stoat_ferret_core/`
- [ ] `mypy src/stoat_ferret tests/` passes
- [ ] `pytest tests/` passes
- [ ] `git diff --exit-code` shows no uncommitted stub changes after fresh generation
