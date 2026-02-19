# 003 Impact Assessment - v007

v007's 9 backlog items (audio mixing, transitions, effect registry, GUI effect workshop, E2E tests) produce 13 documentation impacts across 4 design documents. No MCP tool impacts were found — the auto-dev tools manage the development process, not the product features. No project-specific impact checks are configured (IMPACT_ASSESSMENT.md does not exist).

## Generic Impacts

- **MCP tool_help**: 0 impacts. The v007 changes are product features (audio, transitions, registry, GUI); no MCP tool parameters or behavior will change.
- **MCP tool descriptions**: 0 impacts. Same reasoning as above.
- **Documentation**: 13 impacts across 4 design documents (02-architecture.md, 05-api-specification.md, 01-roadmap.md, 08-gui-architecture.md).

### Affected Documents

| Document | Impact Count | Primary Cause |
|----------|-------------|---------------|
| docs/design/02-architecture.md | 7 | New builders (BL-044, BL-045), registry overhaul (BL-047) |
| docs/design/05-api-specification.md | 4 | New endpoint (BL-046), preview (BL-050), CRUD (BL-051) |
| docs/design/01-roadmap.md | 1 | Phase 2 milestone checkboxes |
| docs/design/08-gui-architecture.md | 1 | Phase 2 GUI milestone checkboxes |

## Project-Specific Impacts

N/A — no IMPACT_ASSESSMENT.md configured.

## Work Items Generated

| Classification | Count | Recommendation |
|---------------|-------|----------------|
| Small | 11 | Consolidate into 2 documentation features (see below) |
| Substantial | 2 | Dedicated documentation features per LRN-030 |
| Cross-version | 0 | All impacts addressable within v007 |

## Recommendations

1. **Create an Architecture Documentation feature** in the theme containing BL-047 (registry). This feature should update 02-architecture.md to reflect the new registry-based dispatch, JSON schema validation, builder protocol, audio/transition builders, and new metrics. Consolidates 7 small impacts (#1-7) plus 1 substantial impact (#2).

2. **Create an API Specification Documentation feature** in the theme containing BL-046 (transition endpoint). This feature should add the /effects/transition endpoint documentation, update the discovery response examples, and replace the "Future" placeholders for preview and CRUD endpoints. Consolidates 3 small impacts (#8-10) plus 1 substantial impact (#11).

3. **Roadmap/GUI checkboxes** (#12, #13) are trivial and can be sub-tasks of either documentation feature above.

4. **Sequencing**: Both documentation features should be placed at the end of their respective themes (after implementation features) so they document the actual implementation rather than the intended design.

## Artifacts

| File | Description |
|------|-------------|
| [impact-table.md](./impact-table.md) | Complete impact table with all 13 impacts |
| [impact-summary.md](./impact-summary.md) | Summary by classification with consolidation recommendation |
