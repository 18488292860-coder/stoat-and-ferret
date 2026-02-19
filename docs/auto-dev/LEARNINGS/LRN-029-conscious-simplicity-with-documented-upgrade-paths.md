## Context

When implementing features with limited initial scope (e.g., 2 effect types), there is a temptation to build abstractions for future extensibility (plugin systems, dispatch tables, separate database tables). However, premature abstraction adds complexity without current value.

## Learning

Choose the simplest implementation that meets current requirements, but document the upgrade path for when complexity is warranted. Plain dict wrappers, JSON columns, and if/elif dispatch are not technical debt when they are conscious trade-offs with documented thresholds for when to refactor. The key distinction: conscious simplicity with a documented trigger ("refactor to registry dispatch when a 3rd effect is added") is a design decision, not an oversight.

## Evidence

In v006 theme 03 (effects-api):
- `EffectRegistry` is a plain dict wrapper — sufficient for 2 effect types
- Effects stored as `effects_json TEXT` column rather than a join table — always read/written as a unit with clip
- `_build_filter_string()` uses if/elif dispatch rather than a plugin system
- Each decision was documented with an explicit upgrade trigger in the retrospective and technical debt section
- All features passed first iteration, confirming the simplicity did not compromise correctness

## Application

When choosing between simple and abstract implementations:
1. Default to the simplest implementation that meets current scope
2. Document the explicit trigger for when to refactor (e.g., "when a 3rd X is added")
3. Record the decision in technical debt tracking so it's visible, not forgotten
4. Distinguish between "conscious simplicity" (documented trade-off) and "accidental simplicity" (oversight)