# Exploration: design-v004-003-impact — Impact Assessment

Impact assessment for v004 identified 14 impacts across documentation, CI/CD, developer experience, and test infrastructure. All 13 backlog items were reviewed. One substantial impact was found; the remaining 13 are small sub-tasks within existing features. No cross-version impacts were identified.

## Generic Impacts

- **Total**: 14 impacts
- **Documentation**: 8 (API spec rewrite for async scan, prototype design update, architecture doc updates, tech stack update, risk assessment examples, search API docs, security audit doc, AGENTS.md commands)
- **CI/CD**: 2 (Rust coverage step, black box test markers)
- **Process Templates**: 2 (requirements template, implementation plan template)
- **Test Infrastructure**: 1 (dependency override migration)
- **Developer Experience**: 1 (AGENTS.md quality gates section)

## Project-Specific Impacts

No project-specific impact checks configured (docs/auto-dev/IMPACT_ASSESSMENT.md not found).

## Work Items Generated

| Classification | Count | Details |
|----------------|-------|---------|
| Substantial | 1 | Scan endpoint API spec rewrite across multiple docs (caused by BL-027) |
| Small | 13 | Sub-tasks within existing backlog features |
| Cross-Version | 0 | All impacts fit within v004 scope |

## Recommendations

### For Logical Design (Task 005)

1. **BL-027 (Async job queue) should include a documentation sub-feature** — The scan endpoint API contract change is the largest single impact. The theme containing BL-027 should explicitly scope updating 05-api-specification.md, 03-prototype-design.md, 02-architecture.md, and 04-technical-stack.md as part of the implementation. This is classified as substantial because it touches 4 design documents and changes the API contract.

2. **Small impacts should be sub-tasks, not separate features** — The 13 small impacts are naturally part of their causing backlog items. During theme design, each feature's implementation plan should include these documentation/CI updates as acceptance criteria or sub-tasks:
   - BL-027 features: Include API doc updates, architecture doc updates, tech stack updates
   - BL-021 features: Include test conftest.py migration
   - BL-009 features: Template updates are the core deliverable (already scoped)
   - BL-010 features: Include CI workflow update
   - BL-023 features: Include CI marker configuration
   - BL-014, BL-026, BL-010: Include AGENTS.md command updates

3. **No new backlog items needed** — All impacts map cleanly to existing backlog items. No gaps require additional scoping.

4. **BL-027 has the highest documentation impact** — 6 of the 14 impacts (43%) are caused by the async scan change. This confirms BL-027 is the most architecturally significant item in v004 despite being P2.

## Impact by Backlog Item

| Backlog Item | Impacts | Notes |
|-------------|---------|-------|
| BL-027 | 6 | Highest impact — async scan changes API contract across multiple docs |
| BL-009 | 2 | Template updates (core deliverable) |
| BL-010 | 2 | CI + AGENTS.md updates |
| BL-021 | 1 | Test infrastructure migration |
| BL-023 | 1 | CI marker configuration |
| BL-014 | 1 | AGENTS.md commands (shared with BL-026, BL-010) |
| BL-016 | 1 | Search API documentation |
| BL-025 | 1 | Net-new security doc |
| BL-026 | 0 | Shared impact with BL-014/BL-010 |
| BL-020 | 0 | No documentation impact — net-new code |
| BL-022 | 0 | No documentation impact — net-new code |
| BL-024 | 0 | No documentation impact — net-new tests |
| BL-012 | 0 | No documentation impact — config changes only |
