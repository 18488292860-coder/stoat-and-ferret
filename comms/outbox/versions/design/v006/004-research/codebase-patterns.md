# Codebase Patterns - v006

## Existing Filter System

### Filter / FilterChain / FilterGraph (v001)

**File:** `rust/stoat_ferret_core/src/ffmpeg/filter.rs`

The current filter system provides three types with builder pattern APIs:

- **`Filter`** - Single FFmpeg filter with key=value parameters. Uses `String` for both keys and values (no expression support). Builder: `Filter::new("name").param("key", value)`.
- **`FilterChain`** - Sequence of filters with optional input/output labels. Builder: `FilterChain::new().input("0:v").filter(f).output("out")`. Labels auto-wrapped in brackets.
- **`FilterGraph`** - Multiple chains joined with semicolons. Builder: `FilterGraph::new().chain(c1).chain(c2)`. Used with `-filter_complex`.

All three implement `Display` for FFmpeg string serialization. All three have PyO3 bindings with `PyRefMut` method chaining pattern.

**Key observations for v006:**
- `Filter.param()` takes `impl ToString` for values, so expression strings can already be passed as values
- No validation of pad labels (BL-038 gap)
- No composition API for branching/merging (BL-039 gap)
- No expression types -- everything is raw strings (BL-037 gap)
- Labels stored as `Vec<String>` without structure (input stream refs vs inter-chain labels are indistinguishable)

### Existing Filter Functions

```rust
pub fn concat(n: usize, v: usize, a: usize) -> Filter
pub fn scale(width: i32, height: i32) -> Filter
pub fn scale_fit(width: i32, height: i32) -> Filter
pub fn pad(width: i32, height: i32, color: &str) -> Filter
pub fn format(pix_fmt: &str) -> Filter
```

Note: `pad()` already uses FFmpeg expressions in parameter values: `x=(ow-iw)/2`, `y=(oh-ih)/2`. This demonstrates that expression strings integrate naturally via the existing `param()` API.

---

## Text Escaping

**File:** `rust/stoat_ferret_core/src/sanitize/mod.rs`

`escape_filter_text()` escapes: `\`, `'`, `:`, `[`, `]`, `;`, `\n`, `\r`. This covers FFmpeg filter argument escaping (levels 2-3). The drawtext builder (BL-040) needs additional `%` escaping for text expansion mode.

**Existing validate_speed()** at line 306: Already validates speed in [0.25, 4.0] range. The speed control builder (BL-041) can delegate to this.

---

## PyO3 Builder Pattern

**Pattern location:** `rust/stoat_ferret_core/src/ffmpeg/filter.rs` lines 129-132

```rust
#[pyo3(name = "param")]
fn py_param(mut slf: PyRefMut<'_, Self>, key: String, value: String) -> PyRefMut<'_, Self> {
    slf.params.push((key, value));
    slf
}
```

This `PyRefMut` return pattern enables method chaining in Python: `filter.param("k1", "v1").param("k2", "v2")`. All v006 builders (expression, drawtext, speed) should follow this pattern per LRN-001.

---

## Proptest Usage

**Files:** `rust/stoat_ferret_core/src/timeline/mod.rs` (line 55), `rust/stoat_ferret_core/src/timeline/range.rs` (line 932)

**Dependency:** `proptest = "1.0"` in `rust/stoat_ferret_core/Cargo.toml` (line 15)

Existing pattern:
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn round_trip_position_integer_fps(frames in 0u64..1_000_000_000) {
        // property assertion
    }

    #[test]
    fn overlap_is_symmetric(
        s1 in 0u64..1000, e1 in 1u64..1001,
        s2 in 0u64..1000, e2 in 1u64..1001,
    ) {
        // property assertion
    }
}
```

**Key observations:**
- Uses inline strategies (range syntax), not derive macro
- `proptest-derive` is NOT a dependency -- manual strategies required
- Testing roundtrip properties (frame->seconds->frame) and symmetry (overlap is symmetric)
- Same patterns applicable to expression serialization roundtrips

---

## Module Organization

**File:** `rust/stoat_ferret_core/src/ffmpeg/mod.rs`

The `ffmpeg` module currently contains `filter.rs` and the command builder. New v006 modules would naturally sit alongside:
- `rust/stoat_ferret_core/src/ffmpeg/expression.rs` (BL-037)
- `rust/stoat_ferret_core/src/ffmpeg/drawtext.rs` (BL-040)
- `rust/stoat_ferret_core/src/ffmpeg/speed.rs` (BL-041)

Validation (BL-038) and composition (BL-039) extend the existing `filter.rs` or sit in adjacent files.

---

## Error Pattern

**File:** `rust/stoat_ferret_core/src/sanitize/mod.rs`

```rust
#[derive(Debug, Clone, PartialEq)]
pub enum BoundsError {
    OutOfRange { name: String, value: f64, min: f64, max: f64 },
    InvalidValue { name: String, value: String, allowed: Vec<String> },
}
```

PyO3 conversion: `map_err(|e| pyo3::exceptions::PyValueError::new_err(e.to_string()))`.

This pattern should be extended for filter graph validation errors (BL-038): `UnconnectedPad { label }`, `CycleDetected { labels }`, `DuplicateLabel { label }`.

---

## DI / create_app Pattern

**File:** `src/stoat_ferret/api/app.py` lines 90-162

```python
def create_app(
    *,
    video_repository: AsyncVideoRepository | None = None,
    project_repository: AsyncProjectRepository | None = None,
    clip_repository: AsyncClipRepository | None = None,
    job_queue: AsyncioJobQueue | None = None,
    ws_manager: ConnectionManager | None = None,
    gui_static_path: str | Path | None = None,
) -> FastAPI:
```

Services stored on `app.state`. When `_deps_injected = True`, lifespan skips DB/worker setup. Dependency functions in routers fall back to real implementations when not injected:

```python
async def get_project_repository(request: Request) -> AsyncProjectRepository:
    repo = getattr(request.app.state, "project_repository", None)
    if repo is not None:
        return repo
    return AsyncSQLiteProjectRepository(request.app.state.db)
```

**v006 implication:** The effect registry (BL-042) should be added as a new `create_app()` kwarg: `effect_registry: EffectRegistry | None = None`.

---

## Clip Model (No Effects Field)

**Python:** `src/stoat_ferret/db/models.py` lines 60-110 - Fields: id, project_id, source_video_id, in_point, out_point, timeline_position, created_at, updated_at. **No effects field.**

**Rust:** `rust/stoat_ferret_core/src/clip/mod.rs` lines 33-139 - Fields: source_path, in_point, out_point, source_duration. **No effects field.**

**API Schema:** `src/stoat_ferret/api/schemas/clip.py` - ClipCreate, ClipUpdate, ClipResponse. **No effects fields.**

**v006 implication:** BL-043 requires adding an `effects: list[EffectConfig]` field to the clip model (Python-side) and corresponding API schemas. The Rust clip does not need an effects field -- effects are stored in Python and generate filter strings via Rust.

---

## Effect Registry (Does Not Exist)

No effect registry, discovery endpoint, or effect registration pattern exists in the codebase. BL-042 must create this from scratch. The existing `register_handler()` pattern from job queue (LRN-009) provides a model: a dict mapping type names to handler functions.

**Recommended pattern:**
```python
class EffectRegistry:
    def __init__(self) -> None:
        self._effects: dict[str, EffectDefinition] = {}

    def register(self, effect_type: str, definition: EffectDefinition) -> None: ...
    def get(self, effect_type: str) -> EffectDefinition | None: ...
    def list_all(self) -> list[EffectDefinition]: ...
```
