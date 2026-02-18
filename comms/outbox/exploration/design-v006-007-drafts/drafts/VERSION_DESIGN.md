# Version Design: v006

## Description

Effects Engine Foundation — Build a greenfield Rust filter expression engine with graph validation, composition system, text overlay builder, speed control builders, effect discovery API, and clip effect application endpoint. Maps to Phase 2, Milestones M2.1–M2.3 from the strategic roadmap.

This is the first version to enter Phase 2 (Effects & Composition), building on the complete Phase 1 foundation (v001–v005).

## Design Artifacts

Full design analysis available at: `comms/outbox/versions/design/v006/`

- `001-environment/` — environment context and readiness checks
- `002-backlog/` — backlog item details (BL-037 through BL-043)
- `003-impact-assessment/` — impact analysis and classification
- `004-research/` — codebase patterns, external research, evidence log
- `005-logical-design/` — original logical design
- `006-critical-thinking/` — refined design, risk assessment, investigation log

## Constraints and Assumptions

- Independence from v005 — pure Rust + API work, no GUI dependency
- PyO3 incremental binding rule — all new Rust types include bindings in the same feature
- Existing `stoat_ferret_core::ffmpeg` module is the integration point for new filter builders
- CI Rust coverage stays at 75% threshold; new code targets >90% individually
- No new external Rust dependencies (no petgraph, no proptest-derive)
- Dev databases are ephemeral — no migration framework needed for schema changes

See `001-environment/version-context.md` for full context.

## Key Design Decisions

- Expression engine modeled as `Expr` enum with `Const`, `Var`, `BinaryOp`, `UnaryOp`, `Func`, `If` variants
- Graph validation via Kahn's algorithm implemented inline (no petgraph dependency)
- Opt-in `validate()` method on FilterGraph — no breaking changes to existing `Display`/`to_string()`
- Clip effects stored as JSON-in-TEXT on Python side only; Rust clip struct unchanged
- Effect registry follows existing DI pattern via `create_app()` kwargs
- Single-value `boxborderw` only; per-side syntax deferred
- Drawtext builder supports both `font()` (fontconfig) and `fontfile()` (explicit path)

See `006-critical-thinking/risk-assessment.md` for rationale.

## Theme Overview

| # | Theme | Goal | Features | Backlog Items |
|---|-------|------|----------|---------------|
| 1 | filter-engine | Foundational Rust filter infrastructure | 3 | BL-037, BL-038, BL-039 |
| 2 | filter-builders | Concrete filter builders for text overlay and speed control | 2 | BL-040, BL-041 |
| 3 | effects-api | Python-side effect registry, discovery API, clip effect API, architecture docs | 3 | BL-042, BL-043, Impact #2 |

See THEME_INDEX.md for details.
