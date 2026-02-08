# Backlog Gap Analysis: v004–v007

The project has 10 open backlog items, but only 1 (BL-003) maps to a planned version. This analysis identified **35 concrete backlog suggestions** across v004–v007: 8 for v004 (testing infrastructure), 9 for v005 (GUI shell), 9 for v006 (effects engine), and 9 for v007 (effect workshop). Key themes: v004 needs black box testing harness and dependency injection; v005 requires an entire frontend project from scratch plus WebSocket support; v006 is a greenfield Rust expression engine; v007 combines remaining effects with a dynamic form-driven GUI.

## Documents

| Document | Description |
|----------|-------------|
| [existing-backlog-mapping.md](existing-backlog-mapping.md) | Maps all 10 open items to planned versions; identifies 5 that naturally fit v004, 3 version-agnostic items, and deferred items from retrospectives never added to backlog |
| [v004-testing-infrastructure.md](v004-testing-infrastructure.md) | M1.8–1.9: 8 gaps, 8 suggestions. Key: InMemory test doubles, DI in create_app(), fixture factory, black box test catalog, FFmpeg contract tests |
| [v005-gui-shell.md](v005-gui-shell.md) | M1.10–1.12: 8 gaps, 9 suggestions. Key: frontend project setup, WebSocket endpoint, application shell, library browser with thumbnails, project manager |
| [v006-effects-engine.md](v006-effects-engine.md) | M2.1–2.3: 9 gaps, 7 suggestions. Key: filter expression engine, graph validation, drawtext builder, speed control, effect discovery API |
| [v007-effect-workshop.md](v007-effect-workshop.md) | M2.4–2.6 + M2.8–2.9: 9 gaps, 9 suggestions. Key: audio mixing, transitions, effect registry, dynamic form generator, live filter preview |

## Summary by Version

| Version | Milestones | Existing Items | Suggested Items | P1 | P2 | P3 |
|---------|-----------|----------------|-----------------|-----|-----|-----|
| v004 | M1.8–1.9 | 0 (5 alignable) | 8 | 4 | 3 | 1 |
| v005 | M1.10–1.12 | 1 (BL-003) | 9 | 5 | 3 | 0 |* |
| v006 | M2.1–2.3 | 0 | 7 | 5 | 2 | 0 |
| v007 | M2.4–2.6, 2.8–2.9 | 0 | 9 | 7 | 1 | 0 |* |

*v005 Item 9 (E2E) is P2; v007 Item 9 (E2E) is P2.

## Cross-Version Dependencies

- v005 depends on v004 black box testing infrastructure (tests validate GUI backend)
- v006 depends on nothing from v005 (pure Rust + API work)
- v007 depends on v006 effects engine and v005 frontend project
- v007 M2.8–2.9 (GUI) depends on v007 M2.4–2.6 (effects) being complete

## Items Needing Exploration (EXP) First

- v005 Item 1: Frontend framework selection (extends BL-003)
- v006 Item 7: Clip effect model design (how effects attach to clips)
- v007 Item 4: Effect registry schema and builder protocol design
- v007 Item 8: Preview thumbnail pipeline (frame extraction + effect application)

## Existing Items to Reassign

5 open backlog items should be tagged for v004:
- BL-009 (property test guidance), BL-010 (Rust coverage), BL-012 (coverage gaps), BL-014 (Docker testing), BL-016 (search behavior unification)
