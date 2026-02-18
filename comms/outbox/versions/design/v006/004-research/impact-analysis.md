# Impact Analysis - v006

## Dependencies (Code/Tools/Configs Affected)

### Rust Crate Changes

| File/Module | Change Type | Reason |
|---|---|---|
| `rust/stoat_ferret_core/src/ffmpeg/expression.rs` | New file | BL-037 expression engine |
| `rust/stoat_ferret_core/src/ffmpeg/drawtext.rs` | New file | BL-040 drawtext builder |
| `rust/stoat_ferret_core/src/ffmpeg/speed.rs` | New file | BL-041 speed builder |
| `rust/stoat_ferret_core/src/ffmpeg/filter.rs` | Modified | BL-038 validation, BL-039 composition |
| `rust/stoat_ferret_core/src/ffmpeg/mod.rs` | Modified | Register new submodules |
| `rust/stoat_ferret_core/src/lib.rs` | Modified | Export new types and PyO3 bindings |
| `rust/stoat_ferret_core/src/sanitize/mod.rs` | Unchanged | Existing `escape_filter_text()` reused by drawtext builder |
| `rust/stoat_ferret_core/Cargo.toml` | Unchanged | proptest already present; no new dependencies needed |

### Python Changes

| File/Module | Change Type | Reason |
|---|---|---|
| Effect registry module (new) | New file | BL-042 effect discovery |
| Effect application endpoint (new) | New file | BL-043 clip effect API |
| Clip/project model | Modified | BL-043 effect storage (schema change) |
| API router | Modified | BL-042, BL-043 new endpoints |
| Python type stubs | Modified | BL-037, BL-039, BL-040, BL-041 PyO3 bindings |

### No External Dependencies Added

- No petgraph (graph validation implemented inline)
- No proptest-derive (manual strategies sufficient)
- No new Python packages

## Breaking Changes

### FilterGraph API (Potentially Breaking)

BL-038 adds validation to `FilterGraph`. If validation is added to the existing `to_string()` method, previously valid (but incorrect) graphs that happen to have unconnected pads would now fail.

**Recommendation:** Add a separate `validate()` method rather than making `to_string()` validate. This preserves backward compatibility. A `validated_to_string()` convenience method can combine both.

### Clip Model Schema Change (Breaking for Existing Projects)

BL-043 adds an effects field to the clip model. Existing project files will not have this field.

**Recommendation:** Use optional field with default empty list. Migration: deserialize without effects field = empty effects list.

## Test Infrastructure Needs

### New Test Files

| Test | Type | Coverage |
|---|---|---|
| Expression roundtrip proptest | Property-based (Rust) | BL-037 AC #4 |
| Expression serialization unit tests | Unit (Rust) | BL-037 AC #3 |
| Graph validation unit tests | Unit (Rust) | BL-038 AC #1-4 |
| Composition unit tests | Unit (Rust) | BL-039 AC #1-4 |
| Drawtext builder unit tests | Unit (Rust) | BL-040 AC #1-3 |
| Drawtext contract tests | Contract (Rust) | BL-040 AC #5 |
| Speed builder unit tests | Unit (Rust) | BL-041 AC #1-5 |
| Effect discovery API tests | Integration (Python) | BL-042 AC #1-5 |
| Effect application API tests | Black box (Python) | BL-043 AC #5 |

### Contract Test Requirements

BL-040 AC #5 specifies "contract tests verify generated commands pass ffmpeg -filter_complex validation." This requires FFmpeg binary availability in the test environment. The existing CI pipeline already has FFmpeg for v001 contract tests.

## Documentation Updates Required

| Document | Update Needed |
|---|---|
| Python type stubs (.pyi) | Auto-generated from PyO3 bindings |
| API specification (05-api-specification.md) | New endpoints for BL-042, BL-043 |
| C4 architecture diagrams | Effects engine component, Rust<->Python boundary |
| CHANGELOG | v006 features summary |
