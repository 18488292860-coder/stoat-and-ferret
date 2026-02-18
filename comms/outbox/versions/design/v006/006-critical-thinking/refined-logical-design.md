# Refined Logical Design — v006 Effects Engine Foundation

## Version Overview

- **Version:** v006
- **Description:** Effects Engine Foundation
- **Roadmap:** Phase 2, Milestones M2.1-M2.3
- **Goal:** Build a greenfield Rust filter expression engine with graph validation, composition system, text overlay builder, speed control builders, effect discovery API, and clip effect application endpoint.
- **Scope:** 7 backlog items (BL-037 through BL-043), organized into 3 themes with 8 features total.
- **Changes from Task 005:** Risk investigation confirmed the logical design is structurally sound. No theme reordering, feature additions, or dependency changes required. Refinements are within individual features (see below).

---

## Theme 01: `01-filter-engine`

**Goal:** Build the foundational Rust filter infrastructure — expression type system, filter graph validation, and composition API.

**Backlog Items:** BL-037, BL-038, BL-039

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-expression-engine` | Type-safe Rust expression builder for FFmpeg filter expressions with proptest validation | BL-037 | None |
| 2 | `002-graph-validation` | Validate FilterGraph pad matching, detect unconnected pads and cycles using Kahn's algorithm | BL-038 | None |
| 3 | `003-filter-composition` | Programmatic chain, branch, and merge composition with automatic pad label management | BL-039 | `002-graph-validation` |

**Risk refinements:**
- **BL-038 validation approach (Risk #2 resolved):** Use opt-in `validate()` method on FilterGraph. Add `validated_to_string()` convenience. Do NOT modify existing `Display`/`to_string()` — zero breaking changes. All existing consumers create valid graphs so no migration needed. See `006-critical-thinking/risk-assessment.md` for evidence.

---

## Theme 02: `02-filter-builders`

**Goal:** Implement concrete filter builders for text overlay and speed control using the expression engine from Theme 01.

**Backlog Items:** BL-040, BL-041

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-drawtext-builder` | Type-safe drawtext filter builder with position presets, styling, and alpha animation via expression engine | BL-040 | Theme 01 `001-expression-engine` |
| 2 | `002-speed-builders` | setpts and atempo filter builders with automatic atempo chaining for speeds above 2.0x | BL-041 | Theme 01 `001-expression-engine` |

**Risk refinements:**
- **Font handling (Risk #4 resolved):** Builder exposes both `font(name)` (fontconfig, cross-platform) and `fontfile(path)` (explicit path) methods. Tests use `font("monospace")`. No platform detection in builder — delegate to FFmpeg.
- **boxborderw (Risk #5 resolved):** Use single-value `boxborderw(width: u32)` only. No pipe-delimited per-side syntax. This avoids FFmpeg version dependency.

---

## Theme 03: `03-effects-api`

**Goal:** Create the Python-side effect registry, discovery API endpoint, clip effect application endpoint, and update architecture documentation.

**Backlog Items:** BL-042, BL-043, plus impact assessment #2

| # | Feature | Goal | Backlog | Dependencies |
|---|---------|------|---------|--------------|
| 1 | `001-effect-discovery` | Effect registry with parameter schemas, AI hints, and GET /effects endpoint | BL-042 | Theme 02 complete |
| 2 | `002-clip-effect-api` | POST endpoint to apply text overlay to clips with effect storage in clip model | BL-043 | `001-effect-discovery` |
| 3 | `003-architecture-docs` | Update 02-architecture.md with new Rust modules, Effects Service, and clip model extension | Impact #2 | `001-effect-discovery`, `002-clip-effect-api` |

**Risk refinements:**
- **Clip effect model (Risk #1 resolved):** Add `effects_json TEXT` column to clips table in `schema.py`. Add `effects: list[dict[str, Any]] | None = None` to Python Clip dataclass. Follow existing `json.dumps()` pattern from `audit.py`. Repository handles JSON serialization. Rust Clip struct unchanged — effects are Python-side business logic. API schemas gain `effects` field on ClipCreate/ClipUpdate/ClipResponse.

---

## Execution Order

### Theme Dependencies

```
Theme 01 (filter-engine) --> Theme 02 (filter-builders) --> Theme 03 (effects-api)
```

Strictly sequential. Unchanged from Task 005.

### Feature Dependencies (detailed)

```
T1-001 (expression-engine) --+---> T2-001 (drawtext) --+--> T3-001 (discovery) --> T3-002 (clip-effect-api) --> T3-003 (architecture-docs)
                              |                         |
T1-002 (graph-validation) --> T1-003 (composition)      |
                                                        |
                              +---> T2-002 (speed) -----+
```

Unchanged from Task 005. Within Theme 01, features 001 and 002 are independent. Within Theme 02, features are independent. Within Theme 03, features are strictly sequential.

---

## Research Sources Adopted

Unchanged from Task 005. See `005-logical-design/logical-design.md` for full list.

---

## Small Impacts (sub-tasks attached to features)

Unchanged from Task 005. See `005-logical-design/logical-design.md` for full list.

---

## Coverage Strategy

Per Risk #3 (Rust coverage ratchet):
- All new v006 Rust modules target >90% coverage individually
- CI threshold remains at 75% (do not ratchet during v006)
- Ratchet to 90% is a separate post-v006 effort
- Each feature's implementation plan should include comprehensive test requirements

---

## Risks and Unknowns (Updated)

| # | Risk | Original Severity | Resolution | Status |
|---|------|-------------------|------------|--------|
| 1 | Clip effect model design | High | Add effects_json TEXT to clips table; Python-only; follow audit.py JSON pattern | Resolved |
| 2 | FilterGraph backward compat | Medium | Opt-in validate() method; no breaking changes to Display/to_string() | Resolved |
| 3 | Rust coverage ratchet | Medium | Keep CI at 75%; new code targets >90%; ratchet deferred | Accepted with mitigation |
| 4 | Font file platform dependency | Low | Builder supports both font() and fontfile(); tests use fontconfig | Accepted with mitigation |
| 5 | boxborderw FFmpeg version | Low | Single-value boxborderw only; defer per-side syntax | Accepted with mitigation |

No remaining TBDs that block v006 execution. All risks have concrete resolutions or mitigations.
