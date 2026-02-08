# Evidence Log — v004

## Concrete Values and Configuration

### Coverage Thresholds
| Parameter | Value | Source | Rationale |
|-----------|-------|--------|-----------|
| Python coverage minimum | 80% | `pyproject.toml:74` | `fail_under = 80` |
| Rust coverage target | 90% | `AGENTS.md:87` | "Rust: 90% minimum" |
| Current Python coverage | 93% | Retrospective insights | v003 achieved 93% |
| llvm-cov baseline | TBD - requires initial measurement | — | No Rust coverage currently tracked |

### Test Infrastructure
| Parameter | Value | Source | Rationale |
|-----------|-------|--------|-----------|
| Existing test count | 395 | Retrospective insights | v003 baseline |
| pytest markers registered | 3 (`slow`, `api`, `contract`) | `pyproject.toml:63-67` | Missing: `blackbox`, `requires_ffmpeg`, `benchmark` |
| asyncio_mode | `auto` | `pyproject.toml:62` | All async tests auto-detected |
| Test matrix | 3 OS × 3 Python = 9 combinations | `.github/workflows/ci.yml:40-43` | ubuntu, windows, macos × 3.10, 3.11, 3.12 |
| FFmpeg available in CI | Yes | `.github/workflows/ci.yml:51-52` | AnimMouse/setup-ffmpeg action |
| FFmpeg-skipped tests | 8 | Retrospective insights | Tests using `requires_ffmpeg` marker |

### Dependency Override Points
| Override | File:Line | Target |
|----------|-----------|--------|
| `get_repository → video_repository` | `tests/test_api/conftest.py:129` | BL-021 migration |
| `get_project_repository → project_repository` | `tests/test_api/conftest.py:130` | BL-021 migration |
| `get_clip_repository → clip_repository` | `tests/test_api/conftest.py:131` | BL-021 migration |
| `get_video_repository → video_repository` | `tests/test_api/conftest.py:132` | BL-021 migration |

### Sanitization Bounds
| Parameter | Range | Source |
|-----------|-------|--------|
| CRF | 0-51 | `sanitize/mod.rs:267` |
| Speed | 0.25-4.0 | `sanitize/mod.rs:306` |
| Volume | 0.0-10.0 | `sanitize/mod.rs:345` |
| Video codecs | 6 whitelisted | `sanitize/mod.rs:359-366` |
| Audio codecs | 4 whitelisted | `sanitize/mod.rs:369` |
| Presets | 10 whitelisted | `sanitize/mod.rs:372-383` |

### Search Behavior
| Implementation | Method | Semantics |
|----------------|--------|-----------|
| `AsyncSQLiteVideoRepository` | FTS5 MATCH | `f'"{query}"*'` — prefix match with tokenization |
| `AsyncInMemoryVideoRepository` | Python `in` | `query_lower in v.filename.lower()` — substring |
| `InMemoryVideoRepository` (sync) | Python `in` | Same substring pattern |

### Missing Dependencies for v004
| Dependency | Required By | Currently Installed |
|------------|-------------|---------------------|
| `hypothesis` | BL-009 (property test guidance) | No |
| `cargo-llvm-cov` | BL-010 (Rust coverage) | No |
| Docker | BL-014 (Docker testing) | Not configured |

### Learnings Verification
| ID | Title | Status | Evidence |
|----|-------|--------|----------|
| LRN-001 | PyO3 Method Chaining with PyRefMut | VERIFIED | FFmpegCommand builder uses this pattern in Rust core |
| LRN-002 | Frame-Accurate Timeline Math | VERIFIED | Position/Duration/FrameRate types exist with u64 frames |
| LRN-003 | Security Validation Whitelist Pattern | VERIFIED | `validate_video_codec`, `validate_preset` etc. use const whitelists |

### Process Template Gaps (BL-009)
| Template | File | Missing Section |
|----------|------|-----------------|
| Requirements | `docs/auto-dev/PROCESS/generic/02-REQUIREMENTS.md` | Property test section |
| Implementation Plan | `docs/auto-dev/PROCESS/generic/03-IMPLEMENTATION-PLAN.md` | Property test planning |

### Scan Operation (BL-027 baseline)
| Parameter | Value | Source |
|-----------|-------|--------|
| Video extensions | `.mp4`, `.mkv`, `.avi`, `.mov`, `.webm`, `.m4v` | `scan.py:13` |
| Current behavior | Synchronous blocking | `scan.py:16-99` |
| Per-file operations | `ffprobe_video()` + repo write | `scan.py:62-86` |
| Job queue timeout | TBD - requires runtime testing | — |

### CI Workflow (baseline for v004 changes)
| Aspect | Current State | v004 Change Needed |
|--------|---------------|-------------------|
| Rust coverage | Not tracked | BL-010: Add llvm-cov step |
| Docker testing | None | BL-014: Add Docker job |
| Marker-based runs | None | BL-023: Add blackbox marker separation |
| Benchmark runs | None | BL-026: Add benchmark command |
