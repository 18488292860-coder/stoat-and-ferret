# Version Context - v004

## Version Goals

**v004 — Testing Infrastructure & Quality Verification**

Build a comprehensive testing and quality verification infrastructure covering black box testing, recording test doubles, contract tests, and quality verification suite. Maps to roadmap milestones M1.8–M1.9.

## Scope

**Estimated:** 8 new items + 5 existing items retagged = 13 total.

## Backlog Items

### New Items (BL-020–027)

| ID | Priority | Title |
|----|----------|-------|
| BL-020 | P1 | Implement InMemory test doubles for projects and jobs |
| BL-021 | P1 | Add dependency injection to create_app() |
| BL-022 | P1 | Build fixture factory with builder pattern |
| BL-023 | P1 | Implement black box test scenario catalog |
| BL-024 | P2 | Contract tests with real FFmpeg |
| BL-025 | P2 | Security audit of Rust sanitization |
| BL-026 | P3 | Rust vs Python performance benchmark |
| BL-027 | P2 | Async job queue for scan operations |

### Existing Items Retagged to v004

| ID | Priority | Title |
|----|----------|-------|
| BL-009 | P2 | Add property test guidance to feature design template |
| BL-010 | P3 | Configure Rust code coverage with llvm-cov |
| BL-012 | P3 | Fix coverage reporting gaps for ImportError fallback |
| BL-014 | P2 | Add Docker-based local testing option |
| BL-016 | P3 | Unify InMemory vs FTS5 search behavior |

## Dependencies

**Internal (within v004):**
- BL-021 depends on BL-020 (DI needs test doubles first)
- BL-022 depends on BL-021 (fixture factory uses DI)
- BL-023 depends on BL-022 (scenario catalog uses fixtures)
- BL-027 depends on BL-020 (async queue needs test doubles)

**Critical path:** BL-020 → BL-021 → BL-022 → BL-023

**External:**
- v004 is a prerequisite for v005 (black box tests validate GUI backend)
- EXP-002 (recording fake pattern) already complete — informs BL-024

## Deferred Items Arriving in v004

| Item | Originally From | Rationale |
|------|----------------|-----------|
| Contract tests with real FFmpeg (BL-024) | M1.3 / v001 | Part of testing infrastructure version |
| Unify InMemory vs FTS5 search behavior (BL-016) | v002 | Testing version is natural fit |
| Async scan job queue (BL-027) | v003 | Deferred from API layer version |

## Constraints and Assumptions

1. **v004 is foundational for v005:** GUI development depends on black box test infrastructure being in place.
2. **No UI work:** v004 is purely backend testing infrastructure — no frontend components.
3. **EXP-002 complete:** The recording fake pattern exploration is done, providing the pattern for FFmpeg test doubles.
4. **Coverage thresholds:** Python 80% minimum, Rust 90% minimum (per AGENTS.md quality gates).
5. **Priority ordering:** P1 items form the critical path; P2/P3 items can be addressed after core infrastructure is in place.

## Version-Agnostic Items (Potentially Addressable)

These are not assigned to v004 but could be addressed opportunistically:
- BL-011 (P3): Consolidate Python/Rust build backends
- BL-018 (P2): Create C4 architecture documentation
- BL-019 (P1): Add Windows bash /dev/null guidance to AGENTS.md
