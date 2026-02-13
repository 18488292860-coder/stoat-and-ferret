# 006 Critical Thinking: v006 — Effects Engine Foundation

Critical thinking review of the v006 logical design investigated all 8 risks from Task 005, resolved 5 through codebase queries and external research, and confirmed 3 with mitigation strategies. No structural design changes were required — all Task 005 assumptions were validated. The design was refined with specific implementation guidance: a bounded expression function whitelist, concrete storage patterns for clip effects, and a specified registry architecture following established codebase patterns.

## Risks Investigated

- **8 total risks** from Task 005
- **5 Investigate now** — resolved with evidence
- **3 Accept with mitigation** — documented with mitigation strategies
- **0 TBD** — no risks require runtime testing to resolve

| Category | Count | Risks |
|----------|-------|-------|
| Investigate now (resolved) | 5 | Clip effects storage, effect registry pattern, expression subset scope, CI FFmpeg, drop-frame timecode |
| Accept with mitigation | 3 | Task 004 research gaps, Rust coverage threshold, BL-043 dependency chain |

## Resolutions

- **Clip effects storage**: `effects_json TEXT` column following audit log JSON pattern. Python owns storage, Rust unchanged (LRN-011). Alembic migration required.
- **Effect registry**: Dictionary-based registry following `AsyncioJobQueue.register_handler()` pattern. Injected via `create_app()` kwargs. Uses Pydantic `.model_json_schema()` for auto JSON Schema generation.
- **Expression whitelist**: Bounded to `{between, if, min, max}` + arithmetic + `{t, PTS, n}`. Driven by actual downstream consumer needs (BL-040 drawtext, BL-041 speed). Extensible for v007.
- **CI FFmpeg**: Fully available on all 9 matrix entries. Existing 3-tier executor infrastructure (Real/Recording/Fake) directly supports new contract tests.
- **Drop-frame timecode**: Confirmed irrelevant — `setpts=PTS/factor` is frame-rate independent. PTS values are linear timestamps unaffected by drop-frame display conventions.

## Design Changes

**No structural changes** to themes, features, or execution order from Task 005. All refinements are scoping clarifications within existing features:

1. **Feature 001-expression-engine**: Added concrete function whitelist (4 required + 5 optional functions)
2. **Feature 002-drawtext-builder**: Confirmed contract test infrastructure and CI availability
3. **Feature 003-speed-control**: Confirmed frame-rate independence with evidence
4. **Feature 001-effect-discovery**: Specified registry pattern using existing job handler registry precedent
5. **Feature 002-clip-effect-model**: Specified storage pattern using existing audit log JSON precedent

## Remaining TBDs

None. All risks either resolved or have documented mitigations. The 3 "accept with mitigation" items are tracking/monitoring concerns, not design unknowns:
- Task 004 research gaps: remaining items are standard FFmpeg doc lookups during implementation
- Rust coverage: track during implementation, add quality-gaps.md if needed post-Theme 02
- BL-043 dependency chain: monitor Theme 03 Feature 001/002 timing

## Confidence Assessment

**High confidence.** The v006 design is robust:
- All 7 backlog items covered with no deferrals
- All high-severity risks resolved with evidence from codebase and external research
- Design uses established codebase patterns (DI, JSON storage, registry, contract tests)
- No circular dependencies, no structural issues discovered
- Execution order validated against confirmed dependency graph

## Output Documents

| Document | Description |
|----------|-------------|
| `risk-assessment.md` | Detailed assessment of all 8 risks with investigation, findings, and resolutions |
| `refined-logical-design.md` | Updated logical design with risk resolutions integrated |
| `investigation-log.md` | Detailed log of all 5 investigations with queries, results, and evidence |
