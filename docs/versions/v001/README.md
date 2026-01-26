# v001 - Foundation Version

**Status:** Complete
**Date:** 2026-01-26

## Summary

Foundation version establishing hybrid Python/Rust architecture for the stoat-and-ferret video editor. This version delivered the core building blocks that all future features depend on.

## What Was Built

### Theme 1: Project Foundation
- Modern Python tooling (uv, ruff, mypy, pytest)
- Rust workspace with PyO3 bindings (abi3-py310)
- GitHub Actions CI pipeline (Ubuntu, Windows, macOS × Python 3.10, 3.11, 3.12)
- Type stubs for IDE support and mypy integration

### Theme 2: Timeline Math (Rust)
- `FrameRate` with rational numerator/denominator representation
- `Position` and `Duration` for frame-accurate timeline positions
- `Clip` with validation (start, end, media_start, media_duration)
- `TimeRange` with half-open interval semantics and set operations
- Property-based tests with proptest

### Theme 3: FFmpeg Command Builder (Rust)
- `FFmpegCommand` fluent builder with input/output management
- `Filter`, `FilterChain`, and `FilterGraph` types
- Input sanitization: text escaping, path validation, bounds checking
- Complete PyO3 bindings with method chaining support

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| u64 frame counts internally | Eliminates floating-point precision drift |
| Rational frame rates | Exact NTSC/PAL representation (e.g., 30000/1001) |
| Result-returning constructors | Enforces valid state invariants at construction |
| Whitelist validation | Prefer allow-listing over block-listing for security |
| PyRefMut method chaining | Fluent APIs work identically in Rust and Python |

## Statistics

| Metric | Value |
|--------|-------|
| Themes | 3 |
| Features | 10 |
| Acceptance Criteria | 42/42 (100%) |
| Rust Tests | 200+ unit + 83 doc tests |
| Python Tests | ~50 integration tests |
| Python Coverage | 86% |

## Artifacts

- Full retrospective: `comms/outbox/versions/execution/v001/retrospective.md`
- Changelog: `docs/CHANGELOG.md`
- Theme retrospectives in `comms/outbox/versions/execution/v001/<theme>/retrospective.md`
