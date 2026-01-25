# Project Backlog

*Last updated: 2026-01-25 14:30*

**Total completed:** 0 | **Cancelled:** 0

## Priority Summary

| Priority | Name | Count |
|----------|------|-------|
| P0 | Critical | 0 |
| P1 | High | 2 |
| P2 | Medium | 1 |
| P3 | Low | 0 |

## Quick Reference

| ID | Pri | Size | Title | Description |
|----|-----|------|-------|-------------|
| <a id="bl-001-ref"></a>[BL-001](#bl-001) | P1 | l | EXP-001: PyO3/maturin hybrid build workflow investigation | Before designing v001, investigate the PyO3/maturin hybri... |
| <a id="bl-002-ref"></a>[BL-002](#bl-002) | P1 | l | EXP-002: Recording fake pattern for FFmpeg executor | Investigate the recording fake pattern for FFmpeg integra... |
| <a id="bl-003-ref"></a>[BL-003](#bl-003) | P2 | m | EXP-003: FastAPI static file serving for GUI | Investigate serving the React/Svelte GUI from FastAPI: |

## Tags Summary

| Tag | Count | Items |
|-----|-------|-------|
| investigation | 3 | BL-001, BL-002, BL-003 |
| v001-prerequisite | 2 | BL-001, BL-002 |
| rust | 1 | BL-001 |
| tooling | 1 | BL-001 |
| testing | 1 | BL-002 |
| ffmpeg | 1 | BL-002 |
| v005-prerequisite | 1 | BL-003 |
| gui | 1 | BL-003 |
| fastapi | 1 | BL-003 |

## Item Details

### P1: High

#### 📋 BL-001: EXP-001: PyO3/maturin hybrid build workflow investigation

**Status:** open
**Tags:** investigation, v001-prerequisite, rust, tooling

Before designing v001, investigate the PyO3/maturin hybrid build chain:

- How does the dev workflow actually function? (edit Rust, rebuild, import in Python)
- What's the compile time impact?
- How do we generate and maintain .pyi stub files?
- What CI configuration is needed for hybrid builds?
- Are there gotchas with maturin develop vs maturin build?

This informs Milestone 1.1 (project setup) and 1.2-1.3 (Rust core).

**Use Case:** This feature addresses: EXP-001: PyO3/maturin hybrid build workflow investigation. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Exploration documents hybrid Python+Rust build workflow
- [ ] Dev experience documented (edit-compile-test cycle)
- [ ] CI setup requirements identified
- [ ] Type stub (.pyi) generation process documented

[↑ Back to list](#bl-001-ref)

#### 📋 BL-002: EXP-002: Recording fake pattern for FFmpeg executor

**Status:** open
**Tags:** investigation, v001-prerequisite, testing, ffmpeg

Investigate the recording fake pattern for FFmpeg integration testing:

- How does RecordingFFmpegExecutor capture commands?
- How do we verify captured commands against expected FFmpeg syntax?
- What's the contract test pattern — run against real FFmpeg periodically?
- How does this integrate with the black box testing harness?

This informs v001 (basic structure) and v004 (full testing infrastructure).

**Use Case:** This feature addresses: EXP-002: Recording fake pattern for FFmpeg executor. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Concrete RecordingFFmpegExecutor implementation documented
- [ ] Recording/playback mechanism for command verification explained
- [ ] Contract test pattern for FFmpeg commands demonstrated
- [ ] Integration with pytest fixtures shown

[↑ Back to list](#bl-002-ref)

### P2: Medium

#### 📋 BL-003: EXP-003: FastAPI static file serving for GUI

**Status:** open
**Tags:** investigation, v005-prerequisite, gui, fastapi

Investigate serving the React/Svelte GUI from FastAPI:

- How do we configure FastAPI to serve static files from Vite build output?
- What's the development workflow — proxy setup for hot reload?
- How do we handle the /gui/* route mounting?
- What about index.html fallback for SPA routing?

This informs v005 (GUI shell).

**Use Case:** This feature addresses: EXP-003: FastAPI static file serving for GUI. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] FastAPI StaticFiles mount configuration documented
- [ ] Vite build output integration explained
- [ ] Development workflow (hot reload) documented
- [ ] Production deployment pattern shown

[↑ Back to list](#bl-003-ref)
