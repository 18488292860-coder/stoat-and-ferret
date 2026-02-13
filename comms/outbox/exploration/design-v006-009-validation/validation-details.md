# Validation Details - v006

## 1. Content Completeness

### Document Pair Comparison (21 pairs)

| # | Document | Result | Notes |
|---|----------|--------|-------|
| 1 | VERSION_DESIGN.md | WARNING | Reformatted by design_version tool. Drops "Key Design Decisions" (5 decisions) and "Design Artifacts" navigation section. Adds boilerplate (Success Criteria checkboxes, Backlog Items links). Core rationale and constraints preserved. |
| 2 | THEME_INDEX.md | WARNING | All 8 feature descriptions replaced with `_Feature description_` placeholders. Theme goals expanded. Adds Path and Notes sections. |
| 3 | Theme 01 THEME_DESIGN.md | MATCH | Identical content |
| 4 | expression-engine requirements.md | MATCH | Identical. FR-001 through FR-005, NFR-001/002, Test Requirements all preserved. |
| 5 | expression-engine implementation-plan.md | MATCH | Identical. All 4 stages, files table, verification commands preserved. |
| 6 | graph-validation requirements.md | MATCH | Identical. FR-001 through FR-006, NFR, Test Requirements preserved. |
| 7 | graph-validation implementation-plan.md | MATCH | Identical. All 4 stages preserved. |
| 8 | Theme 02 THEME_DESIGN.md | MATCH | Identical content |
| 9 | filter-composition requirements.md | MATCH | Identical |
| 10 | filter-composition implementation-plan.md | MATCH | Identical |
| 11 | drawtext-builder requirements.md | MATCH | Identical. FR-001 through FR-007 preserved. |
| 12 | drawtext-builder implementation-plan.md | MATCH | Identical. All 5 stages preserved. |
| 13 | speed-control requirements.md | MATCH | Identical |
| 14 | speed-control implementation-plan.md | MATCH | Identical |
| 15 | Theme 03 THEME_DESIGN.md | MATCH | Identical content |
| 16 | effect-discovery requirements.md | MATCH | Identical. FR-001 through FR-007 preserved. |
| 17 | effect-discovery implementation-plan.md | MATCH | Identical |
| 18 | clip-effect-model requirements.md | MATCH | Identical |
| 19 | clip-effect-model implementation-plan.md | MATCH | Identical |
| 20 | text-overlay-apply requirements.md | MATCH | Identical |
| 21 | text-overlay-apply implementation-plan.md | MATCH | Identical |

**Assessment**: The 2 top-level documents were reformatted by the `design_version` tool (expected behavior). All 19 substantive documents (3 THEME_DESIGN + 8 requirements + 8 implementation plans) preserved with perfect fidelity. No truncation detected in any document.

## 2. Reference Resolution

25 references to design artifacts found across inbox documents:

- **9 references** in THEME_DESIGN.md files (3 per theme): links to `006-critical-thinking/`, `005-logical-design/test-strategy.md`, and `004-research/`
- **8 references** in implementation-plan.md files: all link to `006-critical-thinking/risk-assessment.md`
- **8 references** in requirements.md files: all link to `004-research/`

All 25 resolve to existing files/directories. Zero broken links.

## 3. Notes Propagation

### From backlog-details.md
- BL-037 complexity (foundational piece) → THEME_DESIGN.md for Theme 01
- BL-038 (must not break existing tests) → graph-validation requirements FR-006, implementation-plan Stage 4
- BL-039 depends on BL-038 → filter-composition implementation-plan dependency statement
- BL-040 depends on BL-037 → drawtext-builder implementation-plan dependency statement
- BL-040 contract tests need FFmpeg → drawtext-builder requirements NFR-002, Theme 02 risk table
- BL-041 atempo chaining complexity → speed-control requirements FR-002, implementation-plan Stage 2
- BL-042 depends on BL-040/BL-041 → Theme 03 THEME_DESIGN dependency section
- BL-043 may need investigation → Split into 3 features with dedicated resolution

### From risks-and-unknowns.md (8/8 propagated)
- Clip effects storage model → clip-effect-model requirements FR-002/FR-003, VERSION_DESIGN assumptions
- Effect registry design → effect-discovery requirements FR-006, implementation-plan Stage 1
- Task 004 research incomplete → VERSION_DESIGN assumption, Theme 01 risk table
- Expression subset scope creep → expression-engine requirements FR-003 (whitelist)
- Rust coverage gap → All 5 Rust feature NFRs require >90% module coverage
- BL-043 substantial impacts → 3-feature split in Theme 03
- Contract tests need FFmpeg → drawtext-builder requirements NFR-002
- Drop-frame timecode → speed-control requirements NFR-001, VERSION_DESIGN assumption

### From risk-assessment.md (13/13 propagated)
All resolution details traced to execution documents. Key examples:
- `effects_json TEXT` following audit log pattern → clip-effect-model implementation-plan Stage 1
- Dictionary-based registry following job handler pattern → effect-discovery requirements FR-006
- Expression whitelist `{between, if, min, max}` → expression-engine requirements FR-003
- Record-replay contract test pattern (LRN-008) → drawtext-builder requirements FR-005

## 4. validate_version_design

```json
{
  "valid": true,
  "version": "v006",
  "themes_validated": 3,
  "features_validated": 8,
  "documents": {
    "found": 23,
    "missing": []
  },
  "consistency_errors": []
}
```

## 5. Backlog Alignment

| Backlog Item | Feature(s) | AC Coverage |
|---|---|---|
| BL-037 | 001-expression-engine | 5/5 AC covered |
| BL-038 | 002-graph-validation | 5/5 AC covered |
| BL-039 | 001-filter-composition | 5/5 AC covered |
| BL-040 | 002-drawtext-builder | 5/5 AC covered |
| BL-041 | 003-speed-control | 5/5 AC covered |
| BL-042 | 001-effect-discovery | 5/5 AC covered |
| BL-043 | 002-clip-effect-model + 003-text-overlay-apply | 5/5 AC covered (split) |

All 7 backlog items mapped. All 35 acceptance criteria covered. No orphaned items.

## 6. File Paths

### Pre-existing MODIFY targets (15/15 exist)
- `rust/stoat_ferret_core/src/ffmpeg/mod.rs` — exists
- `rust/stoat_ferret_core/src/lib.rs` — exists
- `stubs/stoat_ferret_core/_core.pyi` — exists
- `rust/stoat_ferret_core/src/ffmpeg/filter.rs` — exists
- `rust/stoat_ferret_core/src/sanitize/mod.rs` — exists
- `src/stoat_ferret/api/app.py` — exists
- `src/stoat_ferret/api/routers/__init__.py` — exists
- `src/stoat_ferret/db/schema.py` — exists
- `src/stoat_ferret/db/models.py` — exists
- `src/stoat_ferret/api/schemas/clip.py` — exists
- `src/stoat_ferret/db/clip_repository.py` — exists
- `src/stoat_ferret/api/websocket/events.py` — exists
- `tests/test_clip_model.py` — exists
- `tests/test_clip_repository_contract.py` — exists
- `tests/test_contract/test_repository_parity.py` — exists

### CREATE targets with missing parent directories
- `rust/stoat_ferret_core/tests/` directory does not exist (affects 5 files: `test_expression.rs`, `test_graph_validation.rs`, `test_composition.rs`, `test_drawtext.rs`, `test_speed.rs`)
- Existing Rust crate uses inline `#[cfg(test)]` modules in source files
- Cargo supports both patterns; directory can be `mkdir`'d during first feature implementation

### All other CREATE targets (11/16) have valid parent directories.

## 7. Dependency Accuracy

### Dependency Graph
```
Theme 01 (no upstream deps):
  F001-expression-engine → (none)
  F002-graph-validation → (none)
  [Parallel execution possible]

Theme 02 (depends on Theme 01):
  F001-filter-composition → T01-F002 (graph-validation)
  F002-drawtext-builder → T01-F001 (expression-engine)
  F003-speed-control → (none, independent)

Theme 03 (depends on Theme 02):
  F001-effect-discovery → T02-F002 + T02-F003 (drawtext + speed)
  F002-clip-effect-model → T03-F001 (effect-discovery)
  F003-text-overlay-apply → T03-F002 + T02-F002 (clip-model + drawtext)
```

Topological sort succeeds. No circular dependencies. All dependency claims are logically verified against implementation details.

## 8. Mitigation Strategy

N/A — v006 is new feature development, not bug fixes. No existing bugs affect the execution path.

## 9. Design Docs Committed

```
$ git status comms/inbox/versions/execution/v006/ --porcelain
(empty — all committed)

$ git status comms/outbox/versions/design/v006/ --porcelain
(empty — all committed)
```

## 10. Handover Document

STARTER_PROMPT.md at `comms/inbox/versions/execution/v006/STARTER_PROMPT.md` contains:
- AGENTS.md reference and PR workflow instruction
- Process steps: THEME_INDEX → theme-by-theme → feature-by-feature
- Status tracking via `comms/outbox/versions/execution/v006/STATUS.md`
- Output document requirements (completion-report, quality-gaps, handoff-to-next)
- Quality gate commands (ruff, mypy, pytest)

## 11. Impact Analysis

All 3 THEME_DESIGN.md files document:
- **Dependencies**: Specific upstream theme/feature references with rationale
- **Dependents**: Downstream theme/feature references
- **Breaking changes**: None — v006 is additive (new Rust modules, new API endpoints, new DB column)
- **Test impact**: Existing tests must continue passing (explicitly stated in graph-validation, clip-effect-model)

VERSION_DESIGN.md documents 7 constraints and 5 assumptions covering cross-cutting concerns.

## 12. Naming Convention

| Pattern | Count | All Match |
|---|---|---|
| Theme folders: `^\d{2}-[a-z][a-z0-9-]*[a-z0-9]$` | 3 | Yes |
| Feature folders: `^\d{3}-[a-z][a-z0-9-]*[a-z0-9]$` | 8 | Yes |
| THEME_INDEX feature lines: `^- \d{3}-[a-z][a-z0-9-]*[a-z0-9]: .+$` | 8 | Yes |
| Double-numbered prefixes: `\d{2,3}-\d{2,3}-` | 0 | N/A (none found) |

## 13. Cross-Reference Consistency

| Source | Themes | Features | Match |
|---|---|---|---|
| THEME_INDEX.md | 3 | 8 | — |
| Disk folders | 3 | 8 | — |
| Index ↔ Disk | 3/3 | 8/8 | Exact (case-sensitive) |

No orphan folders. No phantom index entries.
