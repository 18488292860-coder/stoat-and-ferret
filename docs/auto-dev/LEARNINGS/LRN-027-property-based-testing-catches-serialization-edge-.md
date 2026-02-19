## Context

When implementing types that serialize to string formats (FFmpeg filter expressions, command strings, configuration files), unit tests alone may miss edge cases in the serialization logic — particularly around special values, boundary conditions, and structural invariants.

## Learning

Property-based testing (e.g., proptest in Rust) is high-value for serialization correctness. By generating random valid inputs and checking structural invariants of the output, proptests catch bugs that hand-written unit tests miss. The marginal cost of adding proptests is low once arbitrary generators are written, but the bug-prevention value is high.

## Evidence

In v006, proptest properties on expression and speed control serialization caught real edge cases:
- Balanced parentheses in precedence-aware expression serialization
- No NaN/Inf values in serialized output
- Arity validation rejecting wrong argument counts
- Atempo chain values staying within [0.5, 2.0] quality bounds
- Chain product correctness (individual atempo values multiply to target speed)

These invariants were validated across thousands of random inputs, providing confidence that hand-written unit tests alone could not match.

## Application

When implementing serialization to string formats:
1. Identify structural invariants the output must satisfy (balanced delimiters, no invalid values, valid syntax)
2. Write property-based tests that generate random valid inputs and check these invariants
3. Focus proptests on invariants, not exact output — they complement rather than replace unit tests
4. Invest in arbitrary generators early; they pay dividends across multiple properties