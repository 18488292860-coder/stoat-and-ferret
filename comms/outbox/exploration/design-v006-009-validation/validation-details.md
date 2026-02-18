# Validation Details - v006

## 1. Content Completeness

### Method
Compared all 21 document pairs between Task 007 drafts (`comms/outbox/exploration/design-v006-007-drafts/drafts/`) and persisted inbox documents (`comms/inbox/versions/execution/v006/`).

### Results

| # | Document | Status | Notes |
|---|----------|--------|-------|
| 1 | Theme 01 THEME_DESIGN | MATCH | Identical content |
| 2 | Theme 02 THEME_DESIGN | MATCH | Identical content |
| 3 | Theme 03 THEME_DESIGN | MATCH | Identical content |
| 4 | expression-engine requirements | MATCH | Identical content |
| 5 | expression-engine impl-plan | MATCH | Identical content |
| 6 | graph-validation requirements | MATCH | Identical content |
| 7 | graph-validation impl-plan | MATCH | Identical content |
| 8 | filter-composition requirements | MATCH | Identical content |
| 9 | filter-composition impl-plan | MATCH | Identical content |
| 10 | drawtext-builder requirements | MATCH | Identical content |
| 11 | drawtext-builder impl-plan | MATCH | Identical content |
| 12 | speed-builders requirements | MATCH | Identical content |
| 13 | speed-builders impl-plan | MATCH | Identical content |
| 14 | effect-discovery requirements | MATCH | Identical content |
| 15 | effect-discovery impl-plan | MATCH | Identical content |
| 16 | clip-effect-api requirements | MATCH | Identical content |
| 17 | clip-effect-api impl-plan | MATCH | Identical content |
| 18 | architecture-docs requirements | MATCH | Identical content |
| 19 | architecture-docs impl-plan | MATCH | Identical content |
| 20 | THEME_INDEX.md | DISCREPANCY | Restructured by design_version tool |
| 21 | VERSION_DESIGN.md | DISCREPANCY | Restructured by design_version tool |

### THEME_INDEX.md Discrepancy Detail

**Draft has:** Per-theme `**Backlog Items:**` and `**Dependencies:**` lines, horizontal rule separators.

**Persisted has:** `## Execution Order` header, `**Path:**` lines pointing to inbox folders, expanded `**Goal:**` text, `## Notes` section with execution guidance.

**Content lost:** Per-theme Backlog Items (BL-037/038/039, BL-040/041, BL-042/043/Impact#2) and Dependencies listings. This information is recoverable from THEME_DESIGN files.

### VERSION_DESIGN.md Discrepancy Detail

**Draft has:** `## Design Artifacts` section (6 artifact directories), `## Key Design Decisions` (7 specific decisions), full Backlog Items column in Themes table.

**Persisted has:** `## Backlog Items` as link list, `### Assumptions`, `### Deferred Items`, `## Success Criteria` section.

**Content lost:** Design Artifacts section (pointers to 6 design analysis directories) and Key Design Decisions section (Expr enum modeling, Kahn's algorithm, opt-in validate, JSON-in-TEXT effects, DI pattern, single-value boxborderw, font/fontfile dual support). These are valuable but available in the design artifact store directly.

---

## 2. Reference Resolution

All references in persisted documents point to valid files:

| Reference | Found In | Target Exists |
|-----------|----------|--------------|
| `comms/outbox/versions/design/v006/006-critical-thinking/` | 3 THEME_DESIGNs | Yes (4 files) |
| `comms/outbox/versions/design/v006/004-research/` | 8 requirements.md | Yes (5 files) |

---

## 3. Notes Propagation

All 7 backlog items have `notes: null`. No notes to propagate. Dependency and complexity information from backlog-details.md is structurally captured through theme/feature ordering and explicitly referenced in requirements.

---

## 4. validate_version_design

Tool returned: `valid: true`, 3 themes validated, 8 features validated, 23 documents found, 0 missing, 0 consistency errors.

---

## 5. Backlog Alignment

### Per-Item Mapping

| Backlog Item | Feature | AC Match |
|-------------|---------|----------|
| BL-037 | 001-expression-engine | 5/5 exact |
| BL-038 | 002-graph-validation | 5/5 exact |
| BL-039 | 003-filter-composition | 5/5 exact |
| BL-040 | 001-drawtext-builder | 5/5 exact |
| BL-041 | 002-speed-builders | 5/5 exact |
| BL-042 | 001-effect-discovery | 5/5 exact |
| BL-043 | 002-clip-effect-api | 5/5 exact |

Feature `003-architecture-docs` traces to Impact Assessment #2 (no BL item — correct).

**7/7 backlog items mapped. 35/35 acceptance criteria match.**

---

## 6. File Paths

All 8 implementation plans pass file path validation:

- **Files to modify**: All referenced files exist in the codebase.
- **Files to create**: Parent directories exist (except `src/stoat_ferret/effects/` which is created by 001-effect-discovery as a new Python package).
- **Cross-feature dependencies**: Theme 03 feature 002 references files created by feature 001 — correct sequential ordering ensures they exist.
- **Design artifact references**: All resolve to existing files in `comms/outbox/versions/design/v006/`.

---

## 7. Dependency Accuracy

### Theme Dependencies

| Theme | Stated Dependency | Correct? |
|-------|------------------|----------|
| 01-filter-engine | None | Yes |
| 02-filter-builders | Theme 01 expression-engine | Yes (drawtext uses it; speed-builders are independent but this is conservative/safe) |
| 03-effects-api | Theme 02 | Yes (needs both builders for effect registration) |

### Feature Dependencies

- Theme 01: 001 and 002 are independent; 003 depends on 002 (uses validate()). Sequential execution is valid.
- Theme 02: 001 and 002 are independent (correctly stated).
- Theme 03: 001 -> 002 -> 003 strictly sequential (correctly stated).

### Minor Finding

Theme 02 THEME_DESIGN claims "both builders consume the expression engine." In reality, only drawtext-builder uses Expr types for alpha animation. Speed-builders generate simple formulas without the Expr type system. This is an overstatement but safe (conservative dependency).

No circular dependencies. No missing cross-theme dependencies.

---

## 8. Mitigation Strategy

v006 is greenfield work. Risk assessment documents 5 risks with complete mitigations:

1. **Clip Effect Model Design** — Resolved: `effects_json TEXT` column, ephemeral dev DBs
2. **FilterGraph Backward Compatibility** — Opt-in validate(), no breaking changes
3. **Rust Coverage Ratchet** — Keep 75% threshold, target >90% on new code only
4. **Font File Platform Dependency** — Dual font()/fontfile() API, tests use fontconfig
5. **boxborderw FFmpeg Version** — Single-value only, per-side deferred

No workarounds needed during implementation.

---

## 9. Design Docs Committed

Git status: branch `main`, 0 ahead/0 behind remote. Only modified file is `comms/state/explorations/design-v006-009-validation-*.json` (MCP state, not a design document). All design documents are committed.

---

## 10. Handover Document

STARTER_PROMPT.md is complete with:
- Instructions to read AGENTS.md first
- Process: THEME_INDEX -> themes in order -> features in order
- Status tracking via STATUS.md
- Output document requirements per feature
- Quality gates (ruff check, ruff format, mypy, pytest)

---

## 11. Impact Analysis

Impact table covers 7 areas. Risk assessment covers code-level impacts. Implementation plans list affected test files. No breaking changes — all changes are additive (opt-in validation, new schema column, new endpoints, new types). Test impact assessed for all existing consumers.

---

## 12. Naming Convention

All folder names validated:
- 3 theme folders: `01-filter-engine`, `02-filter-builders`, `03-effects-api` — all match `^\d{2}-[a-z][a-z0-9-]*[a-z0-9]$`
- 8 feature folders — all match `^\d{3}-[a-z][a-z0-9-]*[a-z0-9]$`
- 8 THEME_INDEX feature lines — all match `^- \d{3}-[a-z][a-z0-9-]*[a-z0-9]: .+$`
- 0 double-numbered prefixes (`\d{2,3}-\d{2,3}-`)

---

## 13. Cross-Reference Consistency

| Source | Themes | Features |
|--------|--------|----------|
| THEME_INDEX.md | 3 | 8 |
| Folder structure | 3 | 8 |
| Match | 3/3 | 8/8 |

All names match exactly (case-sensitive). No orphan folders. No missing folders.
