# Version Context — v006

## Version Goals

**v006 — Effects Engine Foundation**

Build the greenfield Rust filter expression engine, graph validation, text overlay, speed control, and effect discovery API. This version covers Phase 2 milestones M2.1–2.3 from the strategic roadmap.

v006 is the first version to tackle video effects — all prior versions (v001–v005) built the foundation (Rust core, database, API, testing, GUI). This version extends the Rust core with a filter expression DSL and composition system.

## Backlog Items

| ID | Priority | Title |
|----|----------|-------|
| BL-037 | P1 | Implement FFmpeg filter expression engine in Rust |
| BL-038 | P1 | Implement filter graph validation |
| BL-039 | P1 | Build filter composition system |
| BL-040 | P1 | Implement drawtext filter builder |
| BL-041 | P1 | Implement speed control filters |
| BL-042 | P2 | Create effect discovery API endpoint |
| BL-043 | P2 | Apply text overlay to clip API |

Full details to be fetched in Task 002.

## Dependency Chain

```
BL-037 (expression engine) ─────────────────────┐
BL-038 (graph validation)  ──→ BL-039 (composition) │
BL-037 ──→ BL-040 (drawtext) ──→ BL-042 (discovery API)
BL-041 (speed control) ─────────→ BL-042 (discovery API)
BL-040 + BL-042 ──→ BL-043 (clip API)
```

- BL-037 and BL-038 are independent starting points
- BL-041 is also independent
- BL-039 depends on BL-038
- BL-040 depends on BL-037
- BL-042 depends on BL-040 and BL-041
- BL-043 depends on BL-040 and BL-042

## Constraints and Assumptions

1. **Independent of v005:** v006 is pure Rust + API work; no GUI changes needed
2. **BL-043 investigation dependency:** BL-043 (clip effect model) has a pending exploration (`BL-043` in Investigation Dependencies table, status: pending). The clip effect model design may need resolution before or during BL-043 implementation.
3. **Coding standards:** All new Rust code must have doc comments, no `unwrap()` in library code, PyO3 bindings added in the same feature (incremental binding rule), `py_` prefix convention for binding methods
4. **Coverage thresholds:** Python 80% minimum, Rust 90% minimum (note: Rust currently at ~75% from v004 deferral)
5. **Python 3.10 compatibility:** Target >=3.10, use `asyncio.TimeoutError` not `builtins.TimeoutError`

## Deferred Items to Be Aware Of

| Item | From | Notes |
|------|------|-------|
| Rust coverage threshold at 75% (target 90%) | v004 | May need attention if v006 Rust additions further dilute coverage |
| SPA fallback routing for deep links | v005 | GUI concern, not relevant to v006 |
| WebSocket connection consolidation | v005 | GUI concern, not relevant to v006 |
| Drop-frame timecode support | v001 | Timeline concern, may intersect with speed control (BL-041) |

## Version-Agnostic Items (Opportunistic)

| ID | Priority | Title |
|----|----------|-------|
| BL-011 | P3 | Consolidate Python/Rust build backends |
| BL-018 | P2 | Create C4 architecture documentation (partially done — full C4 exists) |
| BL-019 | P1 | Add Windows bash /dev/null guidance to AGENTS.md |

## Previously Completed Context

v005 delivered: React GUI shell, library browser, project manager (4 themes, 11 features, 10 backlog items). The system now has a functional web interface over the API layer built in v003, with testing infrastructure from v004.

v006 builds on the Rust core established in v001 and the FFmpeg integration from v002, extending the command builder with a proper filter expression engine.
