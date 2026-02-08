# Existing Backlog Mapping to Planned Versions

## Overview

10 open backlog items exist (BL-003, BL-009–BL-012, BL-014, BL-016, BL-018–BL-019). Only **1** maps to a planned version. The remaining 9 are untagged technical debt from v001–v003 retrospectives.

## Items Mapped to Planned Versions

| ID | Title | Priority | Target Version | Coverage |
|----|-------|----------|---------------|----------|
| BL-003 | EXP-003: FastAPI static file serving for GUI | P2 | v005 | Prerequisite exploration only — no implementation items |

## Unmapped Open Items

| ID | Title | Priority | Suggested Version | Rationale |
|----|-------|----------|-------------------|-----------|
| BL-009 | Add property test guidance to feature design template | P2 | v004 | Testing infrastructure focus aligns with M1.8 |
| BL-010 | Configure Rust code coverage with llvm-cov | P3 | v004 | Quality verification aligns with M1.9 |
| BL-012 | Fix coverage reporting gaps for ImportError fallback | P3 | v004 | Quality verification aligns with M1.9 |
| BL-014 | Add Docker-based local testing option | P2 | v004 | Testing infrastructure aligns with M1.8 |
| BL-016 | Unify InMemory vs FTS5 search behavior | P3 | v004 | Contract test gap — aligns with M1.8 contract tests |
| BL-011 | Consolidate Python/Rust build backends | P3 | Any | Tooling improvement, no version dependency |
| BL-018 | Create C4 architecture documentation | P2 | Any | Documentation, no version dependency |
| BL-019 | Windows bash /dev/null guidance + .gitignore | P1 | Any | Process fix, no version dependency |

## Key Findings

1. **v004 has zero explicit backlog items** despite being the next version. 5 existing tech-debt items naturally align with its testing/quality scope.
2. **v005 has only 1 prerequisite exploration** (BL-003). No implementation items exist for the GUI shell, library browser, or project manager.
3. **v006 and v007 have zero backlog items**. The effects engine and effect workshop are entirely unrepresented.
4. **3 items are version-agnostic** (BL-011, BL-018, BL-019) and could be addressed in any version.

## Deferred Items from v001–v003 Still Pending

These were identified in version retrospectives but never added to the backlog:

- **Contract tests with real FFmpeg** (from v001) — belongs in v004 M1.9
- **Drop-frame timecode support** (from v001) — future enhancement
- **Async job queue for scan operations** (from v003) — belongs in v004 or v005
- **True pagination total count** (from v003) — belongs in v005 (library browser)
- **Stub file consolidation** (from v003) — low priority tooling
