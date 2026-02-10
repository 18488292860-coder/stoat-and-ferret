# C4 Discovery Results - v005

Full-mode discovery of stoat-and-ferret identified 35 source directories across Python (src, tests, scripts, stubs), Rust (rust/stoat_ferret_core/src), and TypeScript/React (gui/src). These are grouped into 4 batches for parallel code-level analysis.

- **Mode:** full
- **Total Source Directories Found:** 35
- **Directories to Process:** 35
- **Directories Unchanged (delta only):** N/A
- **Batch Count:** 4
- **Exclusions Applied:**
  - `.venv` - Python virtual environment
  - `node_modules` - npm dependencies (gui/node_modules)
  - `target` - Rust build artifacts (rust/stoat_ferret_core/target)
  - `__pycache__` - Python bytecode cache
  - `.mypy_cache` - mypy cache
  - `.pytest_cache` - pytest cache
  - `comms/` - MCP communication folders
  - `docs/auto-dev/` - Auto-dev process documentation
  - `docs/ideation/` - Ideation documents
  - `docs/design/` - Design documents (markdown only, no source code)
  - `C4-Documentation` - Not present (first run)
  - `.git` - Version control
  - `gui/node_modules` - Frontend dependencies
  - `rust/stoat_ferret_core/target` - Rust build output

## Existing C4 Documentation

No existing `docs/C4-Documentation/` directory found. This is a first-time full generation.

## Directory Breakdown by Language

| Language | Directories | Files |
|----------|------------|-------|
| Python (src) | 11 | 43 |
| Python (tests) | 9 | 57 |
| Python (scripts) | 1 | 1 |
| Python (stubs) | 1 | 2 |
| Rust | 6 | 13 |
| TypeScript/React | 7 | 51 |
| **Total** | **35** | **167** |
