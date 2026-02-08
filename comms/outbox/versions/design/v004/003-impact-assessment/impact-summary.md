# Impact Summary — v004

## Substantial Impacts (1)

### IMP-1: Scan Endpoint API Specification Rewrite

**Caused by:** BL-027 (Async job queue for scan operations)

The scan endpoint is currently documented across three design documents as a synchronous POST that blocks until completion and returns scan counts. BL-027 changes this to an async pattern where the endpoint returns a job ID immediately and clients poll for status. This requires:

- Rewriting the scan endpoint section in 05-api-specification.md (lines 213-242)
- Documenting the new job status endpoint (GET /jobs/{id}) with full request/response schemas
- Updating the scan section in 03-prototype-design.md (lines 586-605)
- Updating architecture references in 02-architecture.md and 04-technical-stack.md

This is a substantial impact because it affects multiple documents and changes the API contract visible to consumers. It should be scoped as a documentation feature within the BL-027 theme.

---

## Small Impacts (13)

### Documentation Updates

**IMP-2: Job status endpoint API docs** (BL-027)
New GET /jobs/{id} endpoint needs documentation in 05-api-specification.md. Naturally part of BL-027 implementation.

**IMP-3: Prototype design scan section** (BL-027)
03-prototype-design.md scan section needs async pattern update. Brief text change.

**IMP-4: Architecture doc job queue references** (BL-027)
02-architecture.md references arq/Redis for job queue. Actual v004 implementation likely uses in-process queue. Update to reflect reality.

**IMP-14: Technical stack job queue entry** (BL-027)
04-technical-stack.md line 92 lists "Synchronous fake" for job queue. Update after implementation.

**IMP-11: Risk assessment test examples** (BL-027)
06-risk-assessment.md test code examples reference job_queue. Verify alignment after implementation.

**IMP-13: Search API documentation** (BL-016)
05-api-specification.md search behavior may need clarification after InMemory/FTS5 unification.

### Developer Experience Updates

**IMP-5: AGENTS.md quality gates** (BL-014, BL-026, BL-010)
Quality gates section needs new commands: Docker test workflow, benchmark commands, llvm-cov. Three backlog items each add new developer commands not currently documented.

**IMP-6: CI Rust coverage step** (BL-010)
ci.yml needs llvm-cov step. Currently no Rust coverage in CI.

**IMP-7: CI black box test markers** (BL-023)
ci.yml may need pytest marker configuration if black box tests should run separately or be filtered.

### Process & Template Updates

**IMP-8: Requirements template proptest section** (BL-009)
02-REQUIREMENTS.md needs property test section. This is the core deliverable of BL-009.

**IMP-9: Implementation plan template proptest section** (BL-009)
03-IMPLEMENTATION-PLAN.md needs property test planning section. Also core BL-009 deliverable.

### Test Infrastructure Updates

**IMP-10: Test conftest.py dependency override migration** (BL-021)
tests/test_api/conftest.py uses FastAPI dependency_overrides. After BL-021 adds DI to create_app(), these overrides should be migrated to use the new DI parameters. Sub-task of BL-021.

### New Documentation

**IMP-12: Security audit document** (BL-025)
Net-new document in docs/. No existing content to update — part of BL-025 deliverable.

---

## Cross-Version Impacts (0)

No impacts identified that exceed v004 scope. All impacts are addressable within the planned backlog items or as sub-tasks of those items.
