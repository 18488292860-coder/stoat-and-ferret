# Impact Summary - v006

## Small Impacts (sub-task scope)

- **#1 AGENTS.md module list**: Update Rust module structure listing after v006 features add new modules (expressions, graph validation, composition, drawtext, speed). Sub-task of the final Rust feature in the version. *(Caused by: BL-037, BL-038, BL-039, BL-040, BL-041)*

- **#2 Type stub regeneration**: Regenerate `stubs/stoat_ferret_core/_core.pyi` after each feature that adds PyO3 bindings. Already covered by the Incremental Binding Rule in AGENTS.md — each feature must include stub regeneration as a step. *(Caused by: BL-037, BL-039, BL-040, BL-041)*

- **#3 Rust coverage target**: Ensure new Rust code maintains >=90% coverage to close the existing coverage gap (currently 75%). Include coverage checks in quality gates for each Rust feature. *(Caused by: BL-037, BL-038, BL-039, BL-040, BL-041)*

- **#5 Python/Rust boundary description**: Update the responsibility boundary in `02-architecture.md` to include expression construction, graph validation, and filter composition as Rust responsibilities. Small text change, can be paired with impact #4. *(Caused by: BL-037, BL-038, BL-039)*

- **#6 API specification verification**: Verify `05-api-specification.md` accuracy after implementing effect discovery (BL-042) and clip effect application (BL-043). Update only if implementation diverges from spec. *(Caused by: BL-042, BL-043)*

## Substantial Impacts (feature scope)

- **#4+#7 Architecture and C4 documentation update**: The architecture document needs Rust module structure updates, diagram changes, and Effects Service responsibility updates. Combined with the pre-existing C4 documentation gap (not updated since v004), this warrants a dedicated documentation feature that updates both the architecture doc and C4 diagrams to reflect the effects engine. *(Caused by: BL-037, BL-038, BL-039, BL-040, BL-041, BL-042, BL-043)*

- **#8 Clip effect model design**: BL-043 requires an effects storage field on the clip data model that doesn't exist yet. This is a prerequisite design task: define the persistence format for effect configurations, extend the clip schema and repository, and ensure the model supports the effect types being built. LRN-016 specifically warns about this gap. Must be completed before BL-043 implementation begins. *(Caused by: BL-043)*

## Cross-Version Impacts (backlog scope)

None identified. All impacts can be addressed within v006.
