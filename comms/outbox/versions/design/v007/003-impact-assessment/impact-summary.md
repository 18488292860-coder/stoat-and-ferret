# Impact Summary by Classification - v007

## Small Impacts (sub-task scope)

These can be handled as sub-tasks within the feature that causes them.

- **#1** Rust Core Responsibilities list missing audio/transition builders (02-architecture.md) — sub-task of BL-044 or BL-045 feature
- **#3** Rust module structure missing audio/transition modules (02-architecture.md) — sub-task of BL-044 or BL-045 feature
- **#4** PyO3 bindings section missing audio/transition classes (02-architecture.md) — sub-task of BL-044 or BL-045 feature
- **#5** Registered effects table only shows 2 effects (02-architecture.md) — sub-task of BL-047 feature
- **#6** EffectDefinition schema may need updating for registry enhancements (02-architecture.md) — sub-task of BL-047 feature
- **#7** Prometheus metrics list missing effect_applications_total (02-architecture.md) — sub-task of BL-047 feature
- **#8** Discovery response examples only show 2 effects (05-api-specification.md) — sub-task of BL-047 feature
- **#9** Preview endpoint marked "Future/Not yet implemented" (05-api-specification.md) — sub-task of BL-050 feature
- **#10** Update/Delete effect endpoints marked "Future/Not yet implemented" (05-api-specification.md) — sub-task of BL-051 feature
- **#12** Phase 2 roadmap checkboxes (01-roadmap.md) — sub-task of any documentation feature
- **#13** Phase 2 GUI milestone checkboxes (08-gui-architecture.md) — sub-task of any documentation feature

## Substantial Impacts (feature scope)

These require their own dedicated feature in the version plan.

- **#2** Effects Service section rewrite in 02-architecture.md — The registry overhaul (BL-047) fundamentally changes the effects architecture: if/elif dispatch becomes registry-based dispatch, JSON schema validation replaces simple parameter checking, and the builder protocol with DI is a new abstraction. The architecture documentation section (~30 lines) needs a complete rewrite, not a line-by-line edit. Per LRN-030 (Architecture Documentation as Explicit Feature), this should be a dedicated documentation feature within the theme that implements BL-047.
- **#11** New /effects/transition endpoint documentation in 05-api-specification.md — BL-046 adds a new API endpoint category (transition between clips) that has no existing documentation. Documenting this requires a full endpoint section with request/response schemas, validation rules, error cases, and examples. This is substantial enough to be an explicit sub-feature within the theme containing BL-046, or a standalone documentation feature.

## Cross-Version Impacts (backlog scope)

None identified. All impacts can be addressed within v007.

## Consolidation Recommendation

The 11 small impacts across 02-architecture.md and 05-api-specification.md should be consolidated into the 2 substantial documentation features rather than scattered as individual sub-tasks. This creates:

1. **Architecture Documentation Feature**: Covers impacts #1-7 (all 02-architecture.md changes) as a single coherent update to the architecture document after Rust builders and registry are implemented.
2. **API Specification Documentation Feature**: Covers impacts #8-11 (all 05-api-specification.md changes) as a single coherent update to the API spec after all endpoints are implemented.

This aligns with LRN-030 and matches the pattern from v006 T03 F003 where architecture docs were an explicit feature.
