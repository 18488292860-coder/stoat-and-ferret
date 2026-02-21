# Exploration: design-v008-007-drafts

Document drafts for v008 (Startup Integrity & CI Stability) covering 2 themes and 4 features. All documents are lean, referencing the design artifact store at `comms/outbox/versions/design/v008/` for detailed analysis.

## Document Inventory

### Version-Level Documents
- `drafts/manifest.json` — Version metadata, theme/feature numbering, goals (machine-readable)
- `drafts/VERSION_DESIGN.md` — Version-level design overview
- `drafts/THEME_INDEX.md` — Machine-parseable theme/feature index

### Theme 1: application-startup-wiring (3 features)
- `drafts/application-startup-wiring/THEME_DESIGN.md`
- `drafts/application-startup-wiring/database-startup/requirements.md` (BL-058)
- `drafts/application-startup-wiring/database-startup/implementation-plan.md`
- `drafts/application-startup-wiring/logging-startup/requirements.md` (BL-056)
- `drafts/application-startup-wiring/logging-startup/implementation-plan.md`
- `drafts/application-startup-wiring/orphaned-settings/requirements.md` (BL-062)
- `drafts/application-startup-wiring/orphaned-settings/implementation-plan.md`

### Theme 2: ci-stability (1 feature)
- `drafts/ci-stability/THEME_DESIGN.md`
- `drafts/ci-stability/flaky-e2e-fix/requirements.md` (BL-055)
- `drafts/ci-stability/flaky-e2e-fix/implementation-plan.md`

**Total:** 13 documents (1 manifest + 2 version-level + 2 theme designs + 4 requirements + 4 implementation plans)

## Reference Pattern

All documents follow a lean reference pattern:
- VERSION_DESIGN.md references `comms/outbox/versions/design/v008/` for detailed analysis
- THEME_DESIGN.md documents reference `006-critical-thinking/` for risk analysis and `004-research/` for evidence
- Requirements reference `004-research/` for supporting evidence
- Implementation plans reference `006-critical-thinking/risk-assessment.md` for risks
- No design artifact content is duplicated into the drafts

## Completeness Check

All 4 backlog items are covered:

| Backlog | Feature | Requirements | Implementation Plan |
|---------|---------|-------------|---------------------|
| BL-055 (P0) | flaky-e2e-fix | Yes | Yes |
| BL-056 (P1) | logging-startup | Yes | Yes |
| BL-058 (P0) | database-startup | Yes | Yes |
| BL-062 (P2) | orphaned-settings | Yes | Yes |

## Format Verification

- THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+` (verified)
- No numbered lists, bold identifiers, or metadata before colons in feature lines
- No placeholder text in any draft
- All theme/feature slugs are number-prefix-free (MCP tools add prefixes automatically)
- All "Files to Modify" paths verified against codebase structure via `request_clarification`
- Backlog IDs cross-referenced against Task 002 backlog analysis
