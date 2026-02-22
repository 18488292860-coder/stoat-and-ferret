# Design v009 — Document Drafts

Drafted complete design documents for v009 (Observability & GUI Runtime): 2 themes, 6 features. All documents reference the design artifact store at `comms/outbox/versions/design/v009/` rather than duplicating content.

## Document Inventory

| Document | Path |
|----------|------|
| manifest.json | `drafts/manifest.json` |
| VERSION_DESIGN.md | `drafts/VERSION_DESIGN.md` |
| THEME_INDEX.md | `drafts/THEME_INDEX.md` |
| Theme 1 THEME_DESIGN.md | `drafts/observability-pipeline/THEME_DESIGN.md` |
| Feature 001 requirements | `drafts/observability-pipeline/ffmpeg-observability/requirements.md` |
| Feature 001 impl plan | `drafts/observability-pipeline/ffmpeg-observability/implementation-plan.md` |
| Feature 002 requirements | `drafts/observability-pipeline/audit-logging/requirements.md` |
| Feature 002 impl plan | `drafts/observability-pipeline/audit-logging/implementation-plan.md` |
| Feature 003 requirements | `drafts/observability-pipeline/file-logging/requirements.md` |
| Feature 003 impl plan | `drafts/observability-pipeline/file-logging/implementation-plan.md` |
| Theme 2 THEME_DESIGN.md | `drafts/gui-runtime-fixes/THEME_DESIGN.md` |
| Feature 001 requirements | `drafts/gui-runtime-fixes/spa-routing/requirements.md` |
| Feature 001 impl plan | `drafts/gui-runtime-fixes/spa-routing/implementation-plan.md` |
| Feature 002 requirements | `drafts/gui-runtime-fixes/pagination-fix/requirements.md` |
| Feature 002 impl plan | `drafts/gui-runtime-fixes/pagination-fix/implementation-plan.md` |
| Feature 003 requirements | `drafts/gui-runtime-fixes/websocket-broadcasts/requirements.md` |
| Feature 003 impl plan | `drafts/gui-runtime-fixes/websocket-broadcasts/implementation-plan.md` |

**Total:** 1 manifest + 1 version design + 1 theme index + 2 theme designs + 6 requirements + 6 implementation plans = **17 documents**

## Reference Pattern

All documents follow a lean reference pattern:
- VERSION_DESIGN.md references artifact store subdirectories for details
- THEME_DESIGN.md references `006-critical-thinking/` for risk analysis
- requirements.md references `004-research/` for supporting evidence
- implementation-plan.md references specific risk assessment entries

No content from the design artifact store is duplicated in the drafts.

## Completeness Check

All 6 backlog items are covered:
- BL-057 → file-logging (Theme 1, Feature 003)
- BL-059 → ffmpeg-observability (Theme 1, Feature 001)
- BL-060 → audit-logging (Theme 1, Feature 002)
- BL-063 → spa-routing (Theme 2, Feature 001)
- BL-064 → pagination-fix (Theme 2, Feature 002)
- BL-065 → websocket-broadcasts (Theme 2, Feature 003)

## Format Verification

- THEME_INDEX.md uses machine-parseable format: `- NNN-slug: description` (matches `\d{3}-[\w-]+: .+`)
- No numbered lists, bold identifiers, or multi-line entries
- manifest.json uses slug-only names (no number prefixes)
- All folder names use slug-only names (no number prefixes)

## Path Corrections

Two file paths were corrected during verification via `request_clarification`:
1. `src/stoat_ferret/settings.py` → `src/stoat_ferret/api/settings.py` (file-logging)
2. `src/stoat_ferret/services/scan_service.py` → `src/stoat_ferret/api/services/scan.py` (websocket-broadcasts)
