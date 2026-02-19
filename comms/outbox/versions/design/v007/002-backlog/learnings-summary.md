# Learnings Summary - v007

Relevant learnings from the project learning repository (`docs/auto-dev/LEARNINGS/`).

## Directly Applicable to v007

### LRN-001: PyO3 Method Chaining with PyRefMut
Fluent builder patterns for Rust structs exposed to Python using `PyRefMut<'_, Self>`. **v007 applicability:** Audio mixing (BL-044) and transition (BL-045) builders must use this pattern for method chaining in Python. Proven across 5 features in v006.

### LRN-005: Constructor DI Over Dependency Overrides
Inject dependencies via `create_app()` parameters, not `dependency_overrides`. **v007 applicability:** Enhanced effect registry (BL-047) and any new services should use constructor DI. The existing pattern extended cleanly to `effect_registry` in v006.

### LRN-011: Python Business Logic, Rust Input Safety
Rust handles type safety and validation; Python handles business rules and orchestration. **v007 applicability:** Audio/transition parameter validation in Rust, effect registry orchestration and API routing in Python. JSON schema validation boundary: Rust validates parameter ranges, Python validates business constraints.

### LRN-019: Build Infrastructure First
Sequence infrastructure themes before consumer themes. **v007 applicability:** Build Rust builders (BL-044, BL-045) and registry (BL-047) before GUI components (BL-048-051). The dependency chain in v007 aligns with this pattern.

### LRN-024: Focused Zustand Stores Over Monolithic State
Use multiple narrow Zustand stores per feature domain. **v007 applicability:** Effect workshop GUI needs stores for: effect catalog state, form parameter state, preview state, effect stack state. Avoid a single monolithic "effects" store.

### LRN-028: Repeatable Implementation Templates
Establish clear templates in early features; reuse across later features. **v007 applicability:** The first Rust builder (BL-044 or BL-045) sets the template for the other. Similarly, the first GUI component establishes the React component pattern for the rest.

### LRN-029: Conscious Simplicity with Documented Upgrade Paths
Choose simple implementations with explicit refactoring triggers. **v007 applicability:** BL-047 registry design should start simple (dict-based) but must address the v006 refactoring trigger (3rd effect type added). Document the next upgrade path (e.g., plugin system for external effects).

### LRN-031: Detailed Design Specifications Correlate with First-Iteration Success
Specific, testable acceptance criteria correlate with zero re-iterations. **v007 applicability:** All 9 backlog items already have well-specified ACs with action verbs. Design documents should maintain this level of specificity.

## Contextually Relevant

### LRN-026: Opt-In Validation Preserves Backward Compatibility
Add new validation alongside existing behavior. **v007 applicability:** JSON schema validation (BL-047) should be opt-in for existing code paths; new effect types get mandatory validation from the start.

### LRN-030: Architecture Documentation as Explicit Feature
Dedicate a documentation feature to reconcile specs with implementation. **v007 applicability:** Include architecture docs feature in at least one theme, covering effect workshop API, registry schema, and GUI component architecture.

### LRN-022: Automated Accessibility Testing Catches Real Violations
axe-core integration in Playwright catches WCAG violations. **v007 applicability:** BL-052 E2E tests specify WCAG AA compliance for form components. Use axe-core integration from v005 Playwright setup.

### LRN-023: Client-Side Navigation for SPA E2E Tests
Navigate via client-side routing, not direct URL access. **v007 applicability:** Effect workshop E2E tests (BL-052) should navigate through the app, not hit URLs directly, to avoid SPA fallback routing issues.

## Not Applicable to v007

LRN-002 (frame-accurate timeline math), LRN-003 (security whitelist), LRN-004 (deepcopy isolation), LRN-006 (builder dual output), LRN-007 (parity tests), LRN-008 (record-replay), LRN-009 (handler registration), LRN-010 (stdlib asyncio queue), LRN-012 (PyO3 FFI overhead), LRN-013 (progressive coverage), LRN-014 (template-driven process), LRN-015 (single matrix CI), LRN-016 (validate AC against codebase), LRN-017 (empty allowlist), LRN-018 (audit-then-fix), LRN-020 (conditional static mount), LRN-021 (separate vitest config), LRN-025 (feature handoff documents), LRN-027 (property-based testing) — these are either already incorporated into standard practice or not relevant to v007's scope.
