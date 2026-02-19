# Task 006: Learning Extraction — v006

6 new learnings saved (LRN-026 through LRN-031), 3 existing learnings reinforced by new evidence.

## New Learnings

| ID | Title | Tags | Source |
|----|-------|------|--------|
| LRN-026 | Opt-In Validation Preserves Backward Compatibility | pattern, backward-compatibility, validation, api-design | v006/01-filter-engine/002-graph-validation, 01-filter-engine retrospective |
| LRN-027 | Property-Based Testing Catches Serialization Edge Cases | pattern, testing, proptest, serialization, rust | v006/01-filter-engine/001-expression-engine, 02-filter-builders/002-speed-builders, version retrospective |
| LRN-028 | Repeatable Implementation Templates Across Feature Series | process, pattern, templates, scaling, consistency | v006/01-filter-engine retrospective, 02-filter-builders retrospective, version retrospective |
| LRN-029 | Conscious Simplicity with Documented Upgrade Paths | decision-framework, architecture, yagni, simplicity, trade-offs | v006/03-effects-api retrospective, version retrospective |
| LRN-030 | Architecture Documentation as an Explicit Feature Deliverable | process, documentation, architecture, api-design, quality | v006/03-effects-api/003-architecture-docs, 03-effects-api retrospective, version retrospective |
| LRN-031 | Detailed Design Specifications Correlate with First-Iteration Success | process, design, requirements, planning, quality | v006 version retrospective |

## Reinforced Learnings

| ID | Title | New Evidence |
|----|-------|-------------|
| LRN-019 | Build Infrastructure First in Sequential Version Planning | v006's 3-theme dependency chain (expression engine -> builders -> API) completed with zero rework, validating infrastructure-first sequencing across 8 features |
| LRN-001 | PyO3 Method Chaining with PyRefMut | v006 filter composition methods used `PyRefMut<'_, Self>` for mutable FilterGraph operations, confirming the pattern scales to complex composition APIs |
| LRN-005 | Constructor DI over dependency_overrides for FastAPI Testing | v006 theme 03 extended `create_app()` with `effect_registry` kwarg stored on `app.state`, confirming the DI pattern scales cleanly to new services |

## Filtered Out

**12 items filtered** across the following categories:

| Category | Count | Examples |
|----------|-------|---------|
| Too implementation-specific | 6 | FFmpeg positional syntax, atempo chaining algorithm, text escaping layering, clean number formatting, position presets as enum, router prefix design |
| Technical debt tracking | 4 | Label collision risk, expression enum extensibility, stub sync fragility, pre-existing mypy errors |
| Too generic/obvious | 1 | Reuse existing validation rather than duplicating |
| Overlaps with new learning | 1 | JSON column for nested data (covered by LRN-029 conscious simplicity) |
