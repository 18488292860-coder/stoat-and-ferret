# Impact Summary by Classification — v005

## Small Impacts (9)

These can be handled as sub-tasks within existing v005 features. No additional features required.

### Infrastructure
- **#2 — Playwright CI job**: Add E2E CI job as part of BL-036 implementation
- **#5 — App factory static files + WebSocket router**: Add mounts as part of BL-028/BL-029
- **#6 — Lifespan WebSocket manager**: Add startup/shutdown as part of BL-029
- **#11 — gui/ directory scaffolding**: Create directory structure as part of BL-028
- **#12 — Settings expansion**: Add config fields as part of relevant features

### Documentation
- **#7 — Architecture doc update**: Remove "(Future)" from Web UI, add `/ws` and `/gui/*`
- **#8 — Roadmap milestone update**: Mark M1.10-M1.12 after v005 implementation
- **#9 — AGENTS.md quality gates**: Add frontend commands section
- **#10 — Technical stack framework selection**: Update after BL-028 decision

## Substantial Impacts (3)

These require explicit tracking in the version plan as features or dedicated work items.

### #1 — CI Pipeline Extension
- **What**: CI workflow needs Node.js, npm build, Vitest, and Playwright steps. New path filter for `gui/**`. Entirely new job type (E2E with server startup).
- **Why substantial**: CI changes affect all PRs and require careful sequencing — the frontend CI must work before GUI feature PRs can merge. Mistakes here block the entire team.
- **Recommendation**: Include as a feature in the frontend scaffolding theme, implemented after `gui/` setup but before GUI component work begins.

### #3 — Quality Gate Expansion
- **What**: `run_quality_gates` auto-detection needs to recognize `gui/package.json` and run npm-test/tsc. AGENTS.md needs frontend quality commands. The MCP tool already supports npm-test and tsc check types, but project-level integration needs verification.
- **Why substantial**: Quality gates are the safety net for all subsequent features. If frontend checks aren't integrated early, GUI features ship without automated quality verification.
- **Recommendation**: Include as a feature in the frontend scaffolding theme, verified working before any GUI component features.

### #4 — API Specification Updates
- **What**: Three new endpoint types need documentation: thumbnail endpoint (`GET /api/videos/{id}/thumbnail`), WebSocket endpoint (`/ws`), and pagination `total` field verification. The API spec is the contract for frontend development.
- **Why substantial**: The API spec serves as the contract between backend and frontend. Inaccurate specs cause integration failures. The thumbnail endpoint is entirely new, and the WebSocket endpoint introduces a new transport protocol not currently documented.
- **Recommendation**: Update API spec as part of each backend feature (BL-029 adds `/ws` docs, BL-032 adds thumbnail docs, BL-034 verifies `total` docs).

## Cross-Version Impacts (0)

No impacts were identified that exceed v005 scope. All 12 impacts can be addressed within the version.
