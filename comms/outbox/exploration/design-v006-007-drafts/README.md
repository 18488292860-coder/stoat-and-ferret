# Design v006 — Document Drafts (Task 007)

Drafted 22 design documents for v006 Effects Engine Foundation: 3 themes with 8 features total, covering 7 backlog items (BL-037 through BL-043) plus 1 documentation feature (Impact #2). All documents are lean and reference the design artifact store at `comms/outbox/versions/design/v006/`.

## Document Inventory

### Top-Level Documents (3)
- `drafts/manifest.json` — Version metadata, theme/feature numbering, goals (machine-readable)
- `drafts/VERSION_DESIGN.md` — Version-level design overview with artifact store references
- `drafts/THEME_INDEX.md` — Machine-parseable theme/feature index

### Theme Documents (3)
- `drafts/filter-engine/THEME_DESIGN.md` — Theme 01: filter engine infrastructure
- `drafts/filter-builders/THEME_DESIGN.md` — Theme 02: drawtext and speed builders
- `drafts/effects-api/THEME_DESIGN.md` — Theme 03: effect registry and API endpoints

### Feature Documents (16 = 8 requirements + 8 implementation plans)

| Theme | Feature | requirements.md | implementation-plan.md |
|-------|---------|----------------|----------------------|
| filter-engine | expression-engine | BL-037 | 4 stages, 6 files |
| filter-engine | graph-validation | BL-038 | 4 stages, 4 files |
| filter-engine | filter-composition | BL-039 | 4 stages, 4 files |
| filter-builders | drawtext-builder | BL-040 | 4 stages, 6 files |
| filter-builders | speed-builders | BL-041 | 4 stages, 5 files |
| effects-api | effect-discovery | BL-042 | 4 stages, 8 files |
| effects-api | clip-effect-api | BL-043 | 4 stages, 12 files |
| effects-api | architecture-docs | Impact #2 | 2 stages, 2 files |

## Reference Pattern

All documents follow a lean referencing pattern:
- Requirements reference the artifact store: `See comms/outbox/versions/design/v006/004-research/ for supporting evidence`
- Theme designs reference risk assessment: `See comms/outbox/versions/design/v006/006-critical-thinking/`
- Implementation plans reference specific research sections for technical approaches
- No duplication of artifact store content — documents point to the authoritative source

## Completeness Check

| Backlog Item | Feature | Theme | Covered |
|-------------|---------|-------|---------|
| BL-037 | expression-engine | filter-engine | Yes |
| BL-038 | graph-validation | filter-engine | Yes |
| BL-039 | filter-composition | filter-engine | Yes |
| BL-040 | drawtext-builder | filter-builders | Yes |
| BL-041 | speed-builders | filter-builders | Yes |
| BL-042 | effect-discovery | effects-api | Yes |
| BL-043 | clip-effect-api | effects-api | Yes |
| Impact #2 | architecture-docs | effects-api | Yes |

All 7 backlog items are covered. Impact #2 (architecture update) is included as a dedicated feature.

## Format Verification

- THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+` (verified)
- manifest.json is valid JSON with all required fields (verified)
- No theme or feature slug starts with a digit prefix (verified)
- No placeholder text in any draft (verified)

## File Path Verification

All "Files to Modify" paths in implementation plans were verified against the actual codebase structure via Glob queries:
- Rust sources: `rust/stoat_ferret_core/src/ffmpeg/filter.rs`, `mod.rs`, `lib.rs` — all confirmed
- Python sources: `src/stoat_ferret/api/app.py`, `db/schema.py`, `db/models.py`, `db/clip_repository.py`, `api/schemas/clip.py` — all confirmed
- Test files: `tests/test_pyo3_bindings.py`, `tests/test_api/`, `tests/test_blackbox/`, `tests/test_contract/` — all confirmed
- Stubs: `stubs/stoat_ferret_core/_core.pyi` — confirmed
- Docs: `docs/design/02-architecture.md`, `01-roadmap.md`, `05-api-specification.md` — all confirmed
- New files to create have verified parent directories
