## Context

When a version contains multiple features following the same architectural pattern (e.g., multiple Rust types with Python bindings), each feature can either be approached independently or follow an established template from earlier features.

## Learning

Establishing a consistent implementation template in the first feature of a series — and explicitly following it in subsequent features — accelerates implementation and reduces error rates. The template should cover the full lifecycle: implementation, bindings, stubs, tests, and verification. Each successive feature becomes faster as the pattern is internalized.

## Evidence

In v006, the pattern "Rust implementation → PyO3 `#[pymethods]` bindings → stub generation → Python binding tests" was established in 001-expression-engine and followed across all 5 Rust features (expression engine, graph validation, filter composition, drawtext builder, speed control). Theme retrospectives explicitly noted that "each successive feature was faster to implement than the last." All 5 features completed on first iteration with zero quality gate failures. Test count grew steadily (644 → 650 → 664 → 708 → 733) with no churn or regressions.

## Application

When planning a version with multiple features sharing an architectural pattern:
1. Invest extra time in the first feature to establish a clean, explicit template
2. Document the template steps so subsequent features can reference them
3. Include the full lifecycle in the template (implementation, bindings, stubs, tests, verification)
4. Track whether the template is holding up or needs adjustment across features