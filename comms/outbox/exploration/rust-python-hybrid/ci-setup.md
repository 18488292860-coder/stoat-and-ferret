# GitHub Actions CI for PyO3/Maturin Projects

## Quick Start: Generate Workflow

Maturin can generate a complete CI workflow:

```bash
mkdir -p .github/workflows
maturin generate-ci github > .github/workflows/CI.yml
```

This creates a workflow that builds wheels for multiple platforms and Python versions.

## Basic Workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  release:
    types: [published]

permissions:
  contents: read

env:
  PACKAGE_NAME: my_project

jobs:
  # Run tests on multiple platforms
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install Rust
        uses: dtolnay/rust-action@stable
      - name: Build and test
        run: |
          pip install maturin pytest
          maturin develop
          pytest tests/

  # Build wheels for Linux
  linux:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target: [x86_64, aarch64]
    steps:
      - uses: actions/checkout@v4
      - uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.target }}
          manylinux: auto
          args: --release --out dist
      - uses: actions/upload-artifact@v4
        with:
          name: wheels-linux-${{ matrix.target }}
          path: dist

  # Build wheels for Windows
  windows:
    runs-on: windows-latest
    strategy:
      matrix:
        target: [x64, x86]
    steps:
      - uses: actions/checkout@v4
      - uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.target }}
          args: --release --out dist
      - uses: actions/upload-artifact@v4
        with:
          name: wheels-windows-${{ matrix.target }}
          path: dist

  # Build wheels for macOS
  macos:
    runs-on: macos-latest
    strategy:
      matrix:
        target: [x86_64, aarch64]
    steps:
      - uses: actions/checkout@v4
      - uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.target }}-apple-darwin
          args: --release --out dist
      - uses: actions/upload-artifact@v4
        with:
          name: wheels-macos-${{ matrix.target }}
          path: dist

  # Build source distribution
  sdist:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PyO3/maturin-action@v1
        with:
          command: sdist
          args: --out dist
      - uses: actions/upload-artifact@v4
        with:
          name: wheels-sdist
          path: dist

  # Publish to PyPI on release
  release:
    name: Release
    runs-on: ubuntu-latest
    if: github.event_name == 'release' && github.event.action == 'published'
    needs: [test, linux, windows, macos, sdist]
    permissions:
      id-token: write  # For trusted publishing
    steps:
      - uses: actions/download-artifact@v4
        with:
          pattern: wheels-*
          path: dist
          merge-multiple: true
      - uses: pypa/gh-action-pypi-publish@release/v1
```

## maturin-action Configuration

Key options for `PyO3/maturin-action`:

| Option | Description |
|--------|-------------|
| `command` | Maturin subcommand (default: `build`) |
| `args` | Arguments passed to maturin |
| `target` | Cargo target for cross-compilation |
| `manylinux` | Linux platform tag (`auto`, `2014`, `2_28`, etc.) |
| `container` | Custom Docker image |
| `maturin-version` | Pin specific maturin version |

## Manylinux Versions

Choose based on glibc requirements:

| Tag | glibc | Notes |
|-----|-------|-------|
| `manylinux2014` | 2.17 | Minimum for Rust 1.64+ |
| `manylinux_2_28` | 2.28 | Modern distros |
| `musllinux_1_1` | musl | Alpine Linux |

## Trusted Publishing (OIDC)

Configure PyPI for trusted publishing instead of API tokens:

1. Go to PyPI project settings
2. Add new "Trusted Publisher"
3. Enter: repository owner, name, workflow filename
4. Remove the `password` from workflow, use `id-token: write` permission

## Hardened Production Example

For security-critical releases:

```yaml
- uses: PyO3/maturin-action@86b9d133d34bc1b40018696f782949dac11bd380  # Pin by SHA
  with:
    maturin-version: v1.11.5  # Pin version
    args: --release --locked --compatibility pypi
```

## Rust Caching

Speed up builds with caching:

```yaml
- uses: Swatinem/rust-cache@v2
  with:
    key: ${{ matrix.target }}
```

## References

- [maturin-action Repository](https://github.com/PyO3/maturin-action)
- [Maturin Distribution Guide](https://www.maturin.rs/distribution.html)
- [PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/)
