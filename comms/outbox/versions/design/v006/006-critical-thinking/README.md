# 006 - Critical Thinking and Risk Investigation

Critical thinking review of the v006 logical design identified 5 risks from Task 005, investigated all of them via codebase queries and documentation analysis, and resolved or mitigated each. Two risks were fully resolved through investigation (clip effect model design, FilterGraph backward compatibility), and three were accepted with concrete mitigation strategies. The logical design structure — 3 themes, 8 features, strict sequential execution — remains unchanged. Refinements are scoped to individual feature implementation details.

## Risks Investigated

- **5 total risks** from Task 005's logical design
- **2 investigate-now** (resolved via codebase queries): clip effect model, FilterGraph compatibility
- **3 accept-with-mitigation**: Rust coverage ratchet, font platform dependency, boxborderw version

## Resolutions

- **Clip effect model (high):** Effects stored as JSON TEXT column in Python/SQLite only. Rust clip struct unchanged. Follow existing `audit.py` JSON serialization pattern. No migration framework needed.
- **FilterGraph compatibility (medium):** Opt-in `validate()` method preserves backward compatibility. All existing consumers create valid graphs — confirmed by searching every FilterGraph reference in the codebase.

## Design Changes

No structural changes to the logical design from Task 005:
- Same 3 themes, same 8 features, same execution order
- Same backlog coverage (BL-037 through BL-043 + impact #2)
- Refinements are within-feature implementation guidance:
  - BL-038: Use separate `validate()` method, not automatic validation
  - BL-040: Support both `font()` and `fontfile()` methods; single-value `boxborderw` only
  - BL-043: `effects_json TEXT` column following audit.py pattern
  - All Rust code: Target >90% coverage on new modules; keep CI threshold at 75%

## Remaining TBDs

None that block v006 execution. All 5 risks have concrete resolutions or mitigations.

## Confidence Assessment

**High confidence** in the refined design. Key factors:
1. All risks investigated with empirical evidence (file paths, code references, search results)
2. No design-breaking issues discovered — the logical structure is sound
3. Backward compatibility confirmed for the highest-risk item (FilterGraph)
4. Existing codebase patterns (audit.py JSON, DI via create_app) provide clear implementation paths
5. Coverage strategy is pragmatic (new code high coverage, don't block on legacy)
