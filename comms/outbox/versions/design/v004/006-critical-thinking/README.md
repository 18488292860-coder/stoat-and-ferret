# Task 006: Critical Thinking and Risk Investigation — v004

Critical thinking review of the v004 logical design (Task 005) investigated all 8 risks and 5 unknowns. 7 items were resolvable through codebase investigation; 3 require runtime testing; 3 were accepted with existing mitigations. No structural changes to theme groupings, feature ordering, or execution sequence were required. The design is confirmed robust.

## Risks Investigated

- **Total items**: 13 (8 risks, 5 unknowns)
- **Investigated now**: 7 (R1, R2, R3, R4, R6, U3, U5) — all resolved
- **Accepted with mitigation**: 3 (R5, R7, R8) — low risk, clear strategies
- **TBD — runtime testing**: 3 (U1, U2, U4) — cannot be determined pre-implementation

## Resolutions

**R1 (High → Low)**: Scan API breaking change has zero external consumers — only test suite needs migration. Severity downgraded.

**R2 (Medium → Low)**: Domain objects (`Project`, `Video`) are flat dataclasses with scalar-only fields. `copy.deepcopy()` overhead is negligible (microseconds).

**R3 (Medium → Low)**: FTS5 vs InMemory search gap fully characterized. Three specific differences identified: (1) substring vs prefix match, (2) multi-word phrase handling, (3) field scope. Per-token `startswith` addresses the primary gap.

**R4 (Medium → Low scope creep)**: `validate_path` only checks empty/null. Python layer has no directory allowlist. Fix is bounded: ~20 lines adding `ALLOWED_SCAN_ROOTS` config. No Rust changes needed.

**U3 (Unknown → Resolved)**: FakeFFmpegExecutor records args but ignores them. Solution: add optional `strict` mode for contract tests.

**U5 (Unknown → Resolved)**: llvm-cov integrates via single CI matrix combo. Standard `cargo-llvm-cov` with artifact upload.

## Design Changes

**None.** The Task 005 logical design is confirmed without modification. Risk investigation validated that:
- Theme groupings are correct
- Dependency chain is accurate
- Execution order is optimal
- No new features or themes needed

Minor implementation notes were added to the refined design:
- DI migration should note `get_repository`/`get_video_repository` aliasing
- FakeFFmpegExecutor gets `strict` parameter for contract tests
- Search unification has 3 specific behavioral differences to address
- Security audit fix scope is bounded (~20 lines of Python)

## Remaining TBDs

| ID | Item | When Resolved |
|----|------|--------------|
| U1 | Job queue timeout values | BL-027 implementation with real media dirs |
| U2 | Benchmark operation selection | BL-026 implementation with profiling |
| U4 | Docker base image selection | BL-014 implementation with trial builds |

## Confidence Assessment

**High confidence.** The v004 design is well-grounded:
- All 13 backlog items remain mapped with no deferrals
- The highest-severity risk (R1) was fully resolved — no external consumers
- All "Investigate now" items resolved with codebase evidence
- Remaining TBDs are implementation-time decisions, not design blockers
- No circular dependencies or structural issues discovered
- Test strategy covers all risk mitigations
