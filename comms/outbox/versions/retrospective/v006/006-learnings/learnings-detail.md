# Learnings Detail — v006

## LRN-026: Opt-In Validation Preserves Backward Compatibility

**Tags:** pattern, backward-compatibility, validation, api-design
**Source:** v006/01-filter-engine/002-graph-validation completion-report, v006/01-filter-engine retrospective

### Context

When adding validation to an existing type or API that already has consumers (tests, downstream code), the new validation can break existing usage if applied automatically.

### Learning

Make new validation opt-in via explicit method calls rather than automatic on existing operations. Add new methods (`validate()`, `validated_to_string()`) alongside existing methods (`to_string()`) rather than modifying existing behavior. This lets existing consumers continue working unchanged while new consumers can choose stricter validation.

### Evidence

In v006, filter graph validation was added as `validate()` and `validated_to_string()` methods on `FilterGraph`. The existing `Display`/`to_string()` behavior was left completely unchanged. All existing tests (644+) passed unchanged through every subsequent feature that modified `FilterGraph`, including composition APIs that added new methods. Zero backward-compatibility issues across 3 features building on the same type.

### Application

When adding validation, error checking, or stricter behavior to existing types:
1. Add new methods for validated behavior rather than modifying existing ones
2. Keep existing methods with their current (possibly lenient) behavior
3. Document the relationship between strict and lenient methods
4. Let consumers opt into stricter validation at their own pace

---

## LRN-027: Property-Based Testing Catches Serialization Edge Cases

**Tags:** pattern, testing, proptest, serialization, rust
**Source:** v006/01-filter-engine/001-expression-engine completion-report, v006/02-filter-builders/002-speed-builders completion-report, v006 version retrospective

### Context

When implementing types that serialize to string formats (FFmpeg filter expressions, command strings, configuration files), unit tests alone may miss edge cases in the serialization logic — particularly around special values, boundary conditions, and structural invariants.

### Learning

Property-based testing (e.g., proptest in Rust) is high-value for serialization correctness. By generating random valid inputs and checking structural invariants of the output, proptests catch bugs that hand-written unit tests miss. The marginal cost of adding proptests is low once arbitrary generators are written, but the bug-prevention value is high.

### Evidence

In v006, proptest properties on expression and speed control serialization caught real edge cases:
- Balanced parentheses in precedence-aware expression serialization
- No NaN/Inf values in serialized output
- Arity validation rejecting wrong argument counts
- Atempo chain values staying within [0.5, 2.0] quality bounds
- Chain product correctness (individual atempo values multiply to target speed)

These invariants were validated across thousands of random inputs, providing confidence that hand-written unit tests alone could not match.

### Application

When implementing serialization to string formats:
1. Identify structural invariants the output must satisfy (balanced delimiters, no invalid values, valid syntax)
2. Write property-based tests that generate random valid inputs and check these invariants
3. Focus proptests on invariants, not exact output — they complement rather than replace unit tests
4. Invest in arbitrary generators early; they pay dividends across multiple properties

---

## LRN-028: Repeatable Implementation Templates Across Feature Series

**Tags:** process, pattern, templates, scaling, consistency
**Source:** v006/01-filter-engine retrospective, v006/02-filter-builders retrospective, v006 version retrospective

### Context

When a version contains multiple features following the same architectural pattern (e.g., multiple Rust types with Python bindings), each feature can either be approached independently or follow an established template from earlier features.

### Learning

Establishing a consistent implementation template in the first feature of a series — and explicitly following it in subsequent features — accelerates implementation and reduces error rates. The template should cover the full lifecycle: implementation, bindings, stubs, tests, and verification. Each successive feature becomes faster as the pattern is internalized.

### Evidence

In v006, the pattern "Rust implementation -> PyO3 `#[pymethods]` bindings -> stub generation -> Python binding tests" was established in 001-expression-engine and followed across all 5 Rust features (expression engine, graph validation, filter composition, drawtext builder, speed control). Theme retrospectives explicitly noted that "each successive feature was faster to implement than the last." All 5 features completed on first iteration with zero quality gate failures. Test count grew steadily (644 -> 650 -> 664 -> 708 -> 733) with no churn or regressions.

### Application

When planning a version with multiple features sharing an architectural pattern:
1. Invest extra time in the first feature to establish a clean, explicit template
2. Document the template steps so subsequent features can reference them
3. Include the full lifecycle in the template (implementation, bindings, stubs, tests, verification)
4. Track whether the template is holding up or needs adjustment across features

---

## LRN-029: Conscious Simplicity with Documented Upgrade Paths

**Tags:** decision-framework, architecture, yagni, simplicity, trade-offs
**Source:** v006/03-effects-api retrospective, v006 version retrospective

### Context

When implementing features with limited initial scope (e.g., 2 effect types), there is a temptation to build abstractions for future extensibility (plugin systems, dispatch tables, separate database tables). However, premature abstraction adds complexity without current value.

### Learning

Choose the simplest implementation that meets current requirements, but document the upgrade path for when complexity is warranted. Plain dict wrappers, JSON columns, and if/elif dispatch are not technical debt when they are conscious trade-offs with documented thresholds for when to refactor. The key distinction: conscious simplicity with a documented trigger ("refactor to registry dispatch when a 3rd effect is added") is a design decision, not an oversight.

### Evidence

In v006 theme 03 (effects-api):
- `EffectRegistry` is a plain dict wrapper — sufficient for 2 effect types
- Effects stored as `effects_json TEXT` column rather than a join table — always read/written as a unit with clip
- `_build_filter_string()` uses if/elif dispatch rather than a plugin system
- Each decision was documented with an explicit upgrade trigger in the retrospective and technical debt section
- All features passed first iteration, confirming the simplicity did not compromise correctness

### Application

When choosing between simple and abstract implementations:
1. Default to the simplest implementation that meets current scope
2. Document the explicit trigger for when to refactor (e.g., "when a 3rd X is added")
3. Record the decision in technical debt tracking so it's visible, not forgotten
4. Distinguish between "conscious simplicity" (documented trade-off) and "accidental simplicity" (oversight)

---

## LRN-030: Architecture Documentation as an Explicit Feature Deliverable

**Tags:** process, documentation, architecture, api-design, quality
**Source:** v006/03-effects-api/003-architecture-docs completion-report, v006/03-effects-api retrospective, v006 version retrospective

### Context

Architecture documentation often drifts from implementation when treated as an afterthought. API specifications, architecture diagrams, and design documents become outdated as code evolves through feature implementation.

### Learning

Dedicate an explicit feature within a theme to updating architecture documentation after implementation features are complete. This ensures documentation is updated in lockstep with code and receives the same quality gate treatment as implementation features. API specification reconciliation during the documentation feature catches discrepancies between original design and actual implementation.

### Evidence

In v006 theme 03, feature 003-architecture-docs was a dedicated documentation feature that:
- Updated `02-architecture.md` with new Rust modules, Effects Service, and PyO3 bindings
- Reconciled `05-api-specification.md` with actual API implementation, catching discrepancies
- Checked off roadmap milestones M2.1-M2.3
- Passed all 5 acceptance criteria and quality gates
- The reconciliation process discovered that the original API spec didn't match the actual response schema — this would have gone unnoticed without the explicit documentation feature

### Application

For themes that introduce API changes or architectural components:
1. Include a documentation feature as the final feature in the theme
2. Scope it to update architecture docs, API specs, and roadmap status
3. Include API specification reconciliation (comparing spec against actual implementation)
4. Apply the same acceptance criteria and quality gate treatment as code features

---

## LRN-031: Detailed Design Specifications Correlate with First-Iteration Success

**Tags:** process, design, requirements, planning, quality
**Source:** v006 version retrospective

### Context

Feature implementation can require multiple iterations when requirements are ambiguous, acceptance criteria are vague, or implementation plans leave significant design decisions to the implementer.

### Learning

When design documents include detailed acceptance criteria (specific, testable conditions), clear implementation plans, and explicit scope boundaries (what's in and what's out), features are more likely to complete on first iteration without rework. The investment in upfront design specification pays off in reduced iteration count and zero quality gate failures.

### Evidence

In v006, all 8 features across 3 themes completed on first iteration with:
- 40/40 acceptance criteria passed
- 0 quality gate failures
- 0 features requiring re-iteration
- Steady test growth (644 -> 733) with no churn or regressions

Each feature had 5 specific, testable acceptance criteria and a detailed implementation plan. The version retrospective explicitly attributes the first-iteration success to "well-specified design documents" and "detailed acceptance criteria." This pattern was also observed in v005 (referenced in LRN-025) and v004 (referenced in LRN-016), providing cross-version evidence.

### Application

When writing design specifications for features:
1. Write 4-6 specific, testable acceptance criteria per feature (not vague "it should work" criteria)
2. Include implementation plans that name specific files, modules, and patterns to use
3. Define explicit scope boundaries — what's included and what's deferred
4. Reference existing patterns and templates the implementer should follow
