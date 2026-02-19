## Context

Architecture documentation often drifts from implementation when treated as an afterthought. API specifications, architecture diagrams, and design documents become outdated as code evolves through feature implementation.

## Learning

Dedicate an explicit feature within a theme to updating architecture documentation after implementation features are complete. This ensures documentation is updated in lockstep with code and receives the same quality gate treatment as implementation features. API specification reconciliation during the documentation feature catches discrepancies between original design and actual implementation.

## Evidence

In v006 theme 03, feature 003-architecture-docs was a dedicated documentation feature that:
- Updated `02-architecture.md` with new Rust modules, Effects Service, and PyO3 bindings
- Reconciled `05-api-specification.md` with actual API implementation, catching discrepancies
- Checked off roadmap milestones M2.1-M2.3
- Passed all 5 acceptance criteria and quality gates
- The reconciliation process discovered that the original API spec didn't match the actual response schema — this would have gone unnoticed without the explicit documentation feature

## Application

For themes that introduce API changes or architectural components:
1. Include a documentation feature as the final feature in the theme
2. Scope it to update architecture docs, API specs, and roadmap status
3. Include API specification reconciliation (comparing spec against actual implementation)
4. Apply the same acceptance criteria and quality gate treatment as code features