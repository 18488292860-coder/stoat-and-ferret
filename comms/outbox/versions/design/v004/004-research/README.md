# Research & Investigation — v004

Research across 13 backlog items covering test infrastructure, quality verification, security audit, async patterns, and developer experience. Key finding: the codebase already has strong foundations (protocol-based repositories, FFmpeg executor hierarchy, DI via FastAPI Depends) that v004 builds directly upon. The main structural unknowns are resolved: `create_app()` needs parameter injection (not a rewrite), search unification requires FTS5 tokenization in InMemory, and the async job queue should use `asyncio.Queue` (not Redis).

## Research Questions

### Test Infrastructure (BL-020, BL-021, BL-022, BL-023)
1. What InMemory doubles exist and what's missing? → See codebase-patterns.md
2. How does create_app() currently handle DI? → No injectable params; uses `dependency_overrides`
3. What fixture patterns exist? → Inline construction in conftest.py, no builder factory
4. What pytest markers exist? → `slow`, `api`, `contract` registered; no `blackbox` marker

### FFmpeg & Contract Testing (BL-024)
5. How do Recording/Fake executors relate to Real? → Sequential replay, no behavioral verification
6. What verified fake patterns apply? → See external-research.md

### Security (BL-025)
7. What Rust sanitization exists? → `escape_filter_text`, `validate_path`, codec/preset whitelists
8. What attack vectors are untested? → Path traversal (only empty/null checked), symlink following

### Async & Jobs (BL-027)
9. Is scan currently synchronous? → Yes, blocks HTTP request entirely
10. What async patterns exist? → async/await throughout repository layer; no job queue

### Process & DevEx (BL-009, BL-014)
11. Do feature templates have property test sections? → No
12. Does Docker configuration exist? → No Dockerfile or docker-compose.yml exists

### Coverage & Benchmarks (BL-010, BL-012, BL-016, BL-026)
13. What coverage exclusions exist? → `pragma: no cover` on ImportError fallback, `TYPE_CHECKING`
14. Is Rust coverage configured? → No llvm-cov in CI
15. How do InMemory vs FTS5 search differ? → Substring match vs FTS5 MATCH with prefix search

## Findings Summary

- **AsyncInMemoryProjectRepository already exists** at `src/stoat_ferret/db/project_repository.py:184` — BL-020 scope may be smaller than expected (needs deepcopy isolation and seed helpers, but base implementation exists)
- **No InMemoryJobQueue exists** — must be created from scratch for BL-020/BL-027
- **create_app() takes zero parameters** — BL-021 adds optional injectable params
- **4 dependency override points** in test conftest (lines 129-132) — BL-021 should replace these
- **FakeFFmpegExecutor replays sequentially** without validating args match recordings — contract gap for BL-024
- **validate_path() only checks empty and null bytes** — no path traversal or symlink protection in Rust; deferred to Python layer per code comment (line 206)
- **Search semantic gap confirmed**: InMemory uses `str.lower() in str.lower()` substring match; SQLite uses FTS5 `MATCH` with prefix `"query"*`
- **3 learnings verified**: LRN-001 (PyRefMut chaining), LRN-002 (frame-accurate math), LRN-003 (whitelist pattern) — all VERIFIED, patterns still active in codebase

## Unresolved Questions

1. **Job queue timeout values** — TBD, requires runtime testing with real media directories
2. **Benchmark baseline operations** — TBD, need to identify 3 representative Rust vs Python operations at implementation time
3. **llvm-cov threshold** — 90% target per AGENTS.md, but initial baseline unknown
4. **Docker base image** — Python 3.10+ with Rust toolchain; exact image TBD at implementation
5. **FTS5 tokenization in InMemory** — implementing FTS5-like prefix matching in pure Python needs design decision: simple `startswith` per-token or full FTS5 emulation

## Recommendations

1. **BL-020 scope adjustment**: AsyncInMemoryProjectRepository exists; focus on adding deepcopy isolation, seed helpers, and creating InMemoryJobQueue
2. **BL-021 approach**: Add optional params to `create_app()` that default to None (production behavior unchanged); migrate 4 dependency overrides in test conftest
3. **BL-022 builder**: Use dataclass-based builder returning domain objects; `create_via_api()` wraps TestClient POST calls
4. **BL-023 markers**: Add `blackbox` pytest marker; separate from `api` marker
5. **BL-024 contract pattern**: Use verified fakes — parametrized tests running same commands against Real, Recording, and Fake executors
6. **BL-025 security audit**: Focus on path traversal gaps (Rust defers to Python), symlink following, and `escape_filter_text` coverage of all FFmpeg special chars
7. **BL-027 async queue**: Use `asyncio.Queue` with producer-consumer pattern; InMemoryJobQueue for synchronous test mode
8. **Execution order**: BL-020 → BL-021 → BL-022 → BL-023 (strict dependency chain); BL-024, BL-025, BL-027 can parallel after BL-020
