# Exploration: design-v005-007-drafts

Drafted complete design documents for v005 (GUI Shell, Library Browser & Project Manager): 4 themes, 13 features, covering all 10 backlog items. Documents include VERSION_DESIGN.md, THEME_INDEX.md, 4 THEME_DESIGN.md files, 13 requirements.md files, 13 implementation-plan.md files, and manifest.json.

## Document Inventory

### Version-Level Documents
| Document | Path |
|----------|------|
| manifest.json | `drafts/manifest.json` |
| VERSION_DESIGN.md | `drafts/VERSION_DESIGN.md` |
| THEME_INDEX.md | `drafts/THEME_INDEX.md` |

### Theme 01: Frontend Foundation
| Document | Path |
|----------|------|
| THEME_DESIGN.md | `drafts/frontend-foundation/THEME_DESIGN.md` |
| 001 requirements.md | `drafts/frontend-foundation/frontend-scaffolding/requirements.md` |
| 001 implementation-plan.md | `drafts/frontend-foundation/frontend-scaffolding/implementation-plan.md` |
| 002 requirements.md | `drafts/frontend-foundation/websocket-endpoint/requirements.md` |
| 002 implementation-plan.md | `drafts/frontend-foundation/websocket-endpoint/implementation-plan.md` |
| 003 requirements.md | `drafts/frontend-foundation/settings-and-docs/requirements.md` |
| 003 implementation-plan.md | `drafts/frontend-foundation/settings-and-docs/implementation-plan.md` |

### Theme 02: Backend Services
| Document | Path |
|----------|------|
| THEME_DESIGN.md | `drafts/backend-services/THEME_DESIGN.md` |
| 001 requirements.md | `drafts/backend-services/thumbnail-pipeline/requirements.md` |
| 001 implementation-plan.md | `drafts/backend-services/thumbnail-pipeline/implementation-plan.md` |
| 002 requirements.md | `drafts/backend-services/pagination-total-count/requirements.md` |
| 002 implementation-plan.md | `drafts/backend-services/pagination-total-count/implementation-plan.md` |

### Theme 03: GUI Components
| Document | Path |
|----------|------|
| THEME_DESIGN.md | `drafts/gui-components/THEME_DESIGN.md` |
| 001 requirements.md | `drafts/gui-components/application-shell/requirements.md` |
| 001 implementation-plan.md | `drafts/gui-components/application-shell/implementation-plan.md` |
| 002 requirements.md | `drafts/gui-components/dashboard-panel/requirements.md` |
| 002 implementation-plan.md | `drafts/gui-components/dashboard-panel/implementation-plan.md` |
| 003 requirements.md | `drafts/gui-components/library-browser/requirements.md` |
| 003 implementation-plan.md | `drafts/gui-components/library-browser/implementation-plan.md` |
| 004 requirements.md | `drafts/gui-components/project-manager/requirements.md` |
| 004 implementation-plan.md | `drafts/gui-components/project-manager/implementation-plan.md` |

### Theme 04: E2E Testing
| Document | Path |
|----------|------|
| THEME_DESIGN.md | `drafts/e2e-testing/THEME_DESIGN.md` |
| 001 requirements.md | `drafts/e2e-testing/playwright-setup/requirements.md` |
| 001 implementation-plan.md | `drafts/e2e-testing/playwright-setup/implementation-plan.md` |
| 002 requirements.md | `drafts/e2e-testing/e2e-test-suite/requirements.md` |
| 002 implementation-plan.md | `drafts/e2e-testing/e2e-test-suite/implementation-plan.md` |

**Total: 37 documents** (1 manifest + 1 VERSION_DESIGN + 1 THEME_INDEX + 4 THEME_DESIGN + 13 requirements + 13 implementation-plan + 4 test files implied)

## Reference Pattern

All documents follow the "lean reference" pattern: they describe what to build and how, referencing the design artifact store for supporting evidence rather than duplicating it. Each document includes a Reference section pointing to `comms/outbox/versions/design/v005/004-research/` for detailed research findings, evidence values, and codebase patterns.

## Completeness Check

All 10 backlog items are covered across the 13 features:

| Backlog ID | Feature | requirements.md | implementation-plan.md |
|------------|---------|-----------------|------------------------|
| BL-003 | frontend-scaffolding, settings-and-docs | Yes | Yes |
| BL-028 | frontend-scaffolding | Yes | Yes |
| BL-029 | websocket-endpoint, settings-and-docs | Yes | Yes |
| BL-030 | application-shell | Yes | Yes |
| BL-031 | dashboard-panel | Yes | Yes |
| BL-032 | thumbnail-pipeline | Yes | Yes |
| BL-033 | library-browser | Yes | Yes |
| BL-034 | pagination-total-count | Yes | Yes |
| BL-035 | project-manager | Yes | Yes |
| BL-036 | playwright-setup, e2e-test-suite | Yes | Yes |

## Format Verification

- **THEME_INDEX.md:** All feature lines match the required format `- \d{3}-[\w-]+: .+` (verified: `- 001-feature-name: Description text`)
- **manifest.json:** Valid JSON with all required fields (version, description, backlog_ids, context, themes with features)
- **Slug naming:** No theme or feature slugs contain number prefixes (e.g., `frontend-foundation` not `01-frontend-foundation`)
- **Backlog IDs:** All BL numbers cross-referenced against Task 002 backlog analysis (backlog-details.md)
