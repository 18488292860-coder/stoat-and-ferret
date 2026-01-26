# Project Backlog

*Last updated: 2026-01-26 17:16*

**Total completed:** 0 | **Cancelled:** 0

## Priority Summary

| Priority | Name | Count |
|----------|------|-------|
| P0 | Critical | 0 |
| P1 | High | 3 |
| P2 | Medium | 4 |
| P3 | Low | 3 |

## Quick Reference

| ID | Pri | Size | Title | Description |
|----|-----|------|-------|-------------|
| <a id="bl-004-ref"></a>[BL-004](#bl-004) | P1 | m | Expose Clip and ValidationError types to Python | Theme 02 (timeline-math) implemented Clip and ValidationE... |
| <a id="bl-005-ref"></a>[BL-005](#bl-005) | P1 | m | Expose TimeRange and list operations to Python | Theme 02 (timeline-math) implemented TimeRange with set o... |
| <a id="bl-006-ref"></a>[BL-006](#bl-006) | P1 | m | Update AGENTS.md with PyO3 bindings guidance | v001 retrospective identified that deferring PyO3 binding... |
| <a id="bl-003-ref"></a>[BL-003](#bl-003) | P2 | m | EXP-003: FastAPI static file serving for GUI | Investigate serving the React/Svelte GUI from FastAPI: |
| <a id="bl-007-ref"></a>[BL-007](#bl-007) | P2 | m | Automate type stub generation in CI | Type stubs in stubs/stoat_ferret_core/ are manually maint... |
| <a id="bl-008-ref"></a>[BL-008](#bl-008) | P2 | m | Clean up Python API naming inconsistencies | v001 retrospective noted some Python methods have `py_` p... |
| <a id="bl-009-ref"></a>[BL-009](#bl-009) | P2 | m | Add property test guidance to feature design template | v001 retrospective suggested writing proptest invariants ... |
| <a id="bl-010-ref"></a>[BL-010](#bl-010) | P3 | m | Configure Rust code coverage with llvm-cov | v001 retrospective noted Rust code coverage is not tracke... |
| <a id="bl-011-ref"></a>[BL-011](#bl-011) | P3 | m | Consolidate Python/Rust build backends | v001 uses hatchling for Python package management and mat... |
| <a id="bl-012-ref"></a>[BL-012](#bl-012) | P3 | m | Fix coverage reporting gaps for ImportError fallback | v001 retrospective noted ImportError fallback code is exc... |

## Tags Summary

| Tag | Count | Items |
|-----|-------|-------|
| pyo3 | 5 | BL-004, BL-005, BL-006, BL-007, ... |
| rust | 3 | BL-004, BL-005, BL-010 |
| testing | 3 | BL-009, BL-010, BL-012 |
| python-bindings | 2 | BL-004, BL-005 |
| v002 | 2 | BL-004, BL-005 |
| process | 2 | BL-006, BL-009 |
| tooling | 2 | BL-007, BL-011 |
| ci | 2 | BL-007, BL-010 |
| cleanup | 2 | BL-008, BL-012 |
| coverage | 2 | BL-010, BL-012 |
| investigation | 1 | BL-003 |
| v005-prerequisite | 1 | BL-003 |
| gui | 1 | BL-003 |
| fastapi | 1 | BL-003 |
| documentation | 1 | BL-006 |
| type-stubs | 1 | BL-007 |
| api | 1 | BL-008 |
| proptest | 1 | BL-009 |
| build | 1 | BL-011 |
| complexity | 1 | BL-011 |

## Item Details

### P1: High

#### 📋 BL-004: Expose Clip and ValidationError types to Python

**Status:** open
**Tags:** python-bindings, v002, rust, pyo3

Theme 02 (timeline-math) implemented Clip and ValidationError types in Rust but noted "PyO3 bindings not yet added". These types need to be exposed to Python with full type stubs.

**Use Case:** This feature addresses: Expose Clip and ValidationError types to Python. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Clip type exposed to Python with all methods
- [ ] ValidationError exposed with field, message, actual, expected attributes
- [ ] Type stubs updated in stubs/stoat_ferret_core/
- [ ] Integration tests verify round-trip behavior

[↑ Back to list](#bl-004-ref)

#### 📋 BL-005: Expose TimeRange and list operations to Python

**Status:** open
**Tags:** python-bindings, v002, rust, pyo3

Theme 02 (timeline-math) implemented TimeRange with set operations and list operations in Rust but noted these are "not yet exposed to Python". Critical for Python-side timeline manipulation.

**Use Case:** This feature addresses: Expose TimeRange and list operations to Python. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] TimeRange type exposed to Python with half-open interval semantics
- [ ] Set operations (overlap, intersection, union, subtraction, contains) available
- [ ] List operations (find_gaps, merge_ranges) available
- [ ] Type stubs updated
- [ ] Integration tests verify all operations

[↑ Back to list](#bl-005-ref)

#### 📋 BL-006: Update AGENTS.md with PyO3 bindings guidance

**Status:** open
**Tags:** process, documentation, pyo3

v001 retrospective identified that deferring PyO3 bindings created tech debt. AGENTS.md should instruct Claude Code to add Python bindings incrementally in the same feature that implements the Rust types, rather than deferring to a later feature.

**Use Case:** This feature addresses: Update AGENTS.md with PyO3 bindings guidance. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] AGENTS.md includes section on PyO3 bindings best practices
- [ ] Guidance states: add Python bindings in same feature as Rust implementation
- [ ] Guidance states: do not defer bindings to a later feature
- [ ] Example provided showing correct incremental approach

[↑ Back to list](#bl-006-ref)

### P2: Medium

#### 📋 BL-003: EXP-003: FastAPI static file serving for GUI

**Status:** open
**Tags:** investigation, v005-prerequisite, gui, fastapi

Investigate serving the React/Svelte GUI from FastAPI:

- How do we configure FastAPI to serve static files from Vite build output?
- What's the development workflow — proxy setup for hot reload?
- How do we handle the /gui/* route mounting?
- What about index.html fallback for SPA routing?

This informs v005 (GUI shell).

**Use Case:** This feature addresses: EXP-003: FastAPI static file serving for GUI. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] FastAPI StaticFiles mount configuration documented
- [ ] Vite build output integration explained
- [ ] Development workflow (hot reload) documented
- [ ] Production deployment pattern shown

[↑ Back to list](#bl-003-ref)

#### 📋 BL-007: Automate type stub generation in CI

**Status:** open
**Tags:** tooling, ci, pyo3, type-stubs

Type stubs in stubs/stoat_ferret_core/ are manually maintained. This can drift from the actual Rust API. Automate stub generation in CI to catch drift early.

**Use Case:** This feature addresses: Automate type stub generation in CI. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] pyo3-stub-gen or equivalent generates stubs automatically
- [ ] CI step verifies generated stubs match committed stubs
- [ ] CI fails if stubs are out of sync with Rust API
- [ ] Documentation on how to regenerate stubs

[↑ Back to list](#bl-007-ref)

#### 📋 BL-008: Clean up Python API naming inconsistencies

**Status:** open
**Tags:** api, cleanup, pyo3

v001 retrospective noted some Python methods have `py_` prefixes or different names than their Rust counterparts. This creates API inconsistency. Clean up naming to be consistent across languages where possible.

**Use Case:** This feature addresses: Clean up Python API naming inconsistencies. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Audit all Python methods for py_ prefixes
- [ ] Remove unnecessary py_ prefixes where Rust and Python names can match
- [ ] Document any intentional naming differences
- [ ] Update type stubs to match

[↑ Back to list](#bl-008-ref)

#### 📋 BL-009: Add property test guidance to feature design template

**Status:** open
**Tags:** process, testing, proptest

v001 retrospective suggested writing proptest invariants before implementation as executable specifications. Add guidance to feature design templates encouraging this pattern, along with tracking expected test counts.

**Use Case:** This feature addresses: Add property test guidance to feature design template. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Feature design template includes property test section
- [ ] Guidance on writing proptest invariants before implementation
- [ ] Example showing invariant-first design approach
- [ ] Documentation on expected test count tracking

[↑ Back to list](#bl-009-ref)

### P3: Low

#### 📋 BL-010: Configure Rust code coverage with llvm-cov

**Status:** open
**Tags:** testing, coverage, rust, ci

v001 retrospective noted Rust code coverage is not tracked. Configure llvm-cov to measure and report Rust test coverage alongside Python coverage.

**Use Case:** This feature addresses: Configure Rust code coverage with llvm-cov. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] llvm-cov configured for Rust workspace
- [ ] Coverage reports generated during CI
- [ ] Coverage threshold enforced (e.g., 80%)
- [ ] Coverage visible in CI artifacts or dashboard

[↑ Back to list](#bl-010-ref)

#### 📋 BL-011: Consolidate Python/Rust build backends

**Status:** open
**Tags:** tooling, build, complexity

v001 uses hatchling for Python package management and maturin for Rust/PyO3 builds. This dual-backend approach adds complexity. Evaluate whether the build system can be simplified.

**Use Case:** This feature addresses: Consolidate Python/Rust build backends. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Evaluate if hatchling + maturin can be unified
- [ ] Document build system architecture and rationale
- [ ] Simplify if possible without breaking functionality
- [ ] Update developer documentation

[↑ Back to list](#bl-011-ref)

#### 📋 BL-012: Fix coverage reporting gaps for ImportError fallback

**Status:** open
**Tags:** testing, coverage, cleanup

v001 retrospective noted ImportError fallback code is excluded from coverage. Review all coverage exclusions and ensure they are intentional and documented.

**Use Case:** This feature addresses: Fix coverage reporting gaps for ImportError fallback. It improves the system by resolving the described requirement.

**Acceptance Criteria:**
- [ ] Identify all coverage exclusions in Python code
- [ ] Remove or justify each exclusion
- [ ] ImportError fallback properly tested or documented as intentional exclusion
- [ ] Coverage threshold maintained

[↑ Back to list](#bl-012-ref)
