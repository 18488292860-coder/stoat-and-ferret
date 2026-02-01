# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## v003 - 2026-01-28

### Added
- FastAPI REST API application with lifespan management
- Async repository layer (AsyncVideoRepository, AsyncProjectRepository, AsyncClipRepository) using aiosqlite
- Video library API endpoints (list, detail, search, scan, delete)
- Project and Clip data models with Rust validation
- Projects and clips API endpoints
- Health endpoints (/health/liveness, /health/readiness)
- Correlation ID middleware for request tracing
- Prometheus metrics middleware
- Externalized configuration with pydantic-settings
- CI path filters to skip heavy steps for docs-only commits
- Migration verification in CI (upgrade/downgrade/upgrade cycle)

### Changed
- Repository pattern now supports both sync and async implementations

### Fixed
- N/A

## v002 - 2026-01-27

### Added
- SQLite database with Alembic migrations
- Video repository pattern with sync implementation
- FFmpeg executor with dependency injection
- Observability infrastructure (structlog, Prometheus)
- PyO3 bindings for Clip and ValidationError types
- PyO3 bindings for TimeRange with set operations
- Type stub generation automation in CI
- Python API naming consistency improvements

### Changed
- Enhanced Rust-Python integration with complete type coverage

### Fixed
- N/A

## v001 - 2026-01-26

### Added
- Project setup with Python + Rust (PyO3/maturin) hybrid build
- Rust core: Timecode type with frame-accurate calculations
- Rust core: Timeline math (Clip, TimeRange validation)
- Rust core: FFmpeg command builder
- PyO3 bindings with type stubs
- CI/CD pipeline with GitHub Actions
- Test infrastructure (pytest, proptest)
- Project documentation structure

### Changed
- N/A

### Fixed
- N/A
