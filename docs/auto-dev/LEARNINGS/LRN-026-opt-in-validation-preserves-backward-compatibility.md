## Context

When adding validation to an existing type or API that already has consumers (tests, downstream code), the new validation can break existing usage if applied automatically.

## Learning

Make new validation opt-in via explicit method calls rather than automatic on existing operations. Add new methods (`validate()`, `validated_to_string()`) alongside existing methods (`to_string()`) rather than modifying existing behavior. This lets existing consumers continue working unchanged while new consumers can choose stricter validation.

## Evidence

In v006, filter graph validation was added as `validate()` and `validated_to_string()` methods on `FilterGraph`. The existing `Display`/`to_string()` behavior was left completely unchanged. All existing tests (644+) passed unchanged through every subsequent feature that modified `FilterGraph`, including composition APIs that added new methods. Zero backward-compatibility issues across 3 features building on the same type.

## Application

When adding validation, error checking, or stricter behavior to existing types:
1. Add new methods for validated behavior rather than modifying existing ones
2. Keep existing methods with their current (possibly lenient) behavior
3. Document the relationship between strict and lenient methods
4. Let consumers opt into stricter validation at their own pace