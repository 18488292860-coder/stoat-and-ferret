# 004 - Research and Investigation

Research for v006 effects engine covered five domains: FFmpeg filter expressions, drawtext filter parameters, atempo chaining mechanics, Rust proptest for property-based testing, and graph validation algorithms for DAG-based filter graphs. All research questions from the 7 backlog items (BL-037 through BL-043) were investigated. Key findings: FFmpeg expressions have a well-defined grammar with 30+ functions suitable for type-safe Rust modeling; drawtext has 20+ parameters with expression-capable x/y/alpha fields; atempo accepts [0.5, 100.0] but degrades above 2.0 requiring chaining; proptest already exists in the codebase with patterns ready to extend; and Kahn's algorithm is the recommended approach for filter graph cycle detection.

## Research Questions

1. **FFmpeg expression syntax** (BL-037): What functions, variables, operators, and grammar does FFmpeg's expression evaluator support?
2. **drawtext filter parameters** (BL-040): What are all positioning, font, shadow, box, and alpha parameters? How do expressions work in x/y/alpha?
3. **atempo chaining** (BL-041): What is the valid range? How to chain for speeds above 2.0x or below 0.5x? What is the setpts formula?
4. **Proptest for Rust** (BL-037, BL-040, BL-041): How to generate arbitrary enum/struct instances? Regex string generation? Best practices for expression builders?
5. **Graph validation** (BL-038): Topological sort vs DFS for cycle detection? Unconnected node detection? Pad label matching?

## Findings Summary

- **Expression engine (BL-037)**: FFmpeg expressions support 35+ functions (arithmetic, comparison, conditional, trigonometric, bitwise), 10 internal variable slots, and constants (PI, E, PHI). The expression grammar maps cleanly to a Rust enum hierarchy. The `between(t,start,end)` pattern is the core building block for enable/alpha expressions.
- **drawtext (BL-040)**: 20+ parameters including x, y, fontsize, fontcolor, fontfile, alpha, box, boxcolor, boxborderw, shadowx, shadowy, shadowcolor, fontcolor_expr, enable. Expression variables in x/y include: w, h, text_w (tw), text_h (th), line_h (lh), main_w (W), main_h (H), dar, sar, n, t, max_glyph_a, max_glyph_d, max_glyph_h, max_glyph_w. Text escaping requires handling `\`, `'`, `:`, `[`, `]`, `;`, `%`.
- **Speed control (BL-041)**: atempo range is [0.5, 100.0] but quality degrades above 2.0 (sample skipping instead of blending). Chaining formula: factor the desired tempo into values within [0.5, 2.0] and chain filters. setpts formula: `PTS/speed` for speedup (e.g., `PTS*0.5` = 2x speed), `PTS*factor` for slowdown.
- **Proptest (BL-037)**: `#[derive(Arbitrary)]` for enums/structs, `prop_oneof!` for custom variant strategies, `prop_compose!` for building complex strategies, regex-based string generation with `#[proptest(regex = "...")]`. Already used in timeline module.
- **Graph validation (BL-038)**: Kahn's algorithm (BFS + in-degree tracking) detects cycles in O(V+E) and produces topological ordering simultaneously. petgraph crate provides `toposort()` and `is_cyclic_directed()` but may be unnecessary -- a minimal in-module implementation is simpler for the FilterGraph use case.

## Learning Verification

| Learning | Status | Evidence |
|----------|--------|----------|
| LRN-001 | VERIFIED | `PyRefMut<'_, Self>` pattern used in 14 methods in `command.rs` and 5 methods in `filter.rs` |
| LRN-005 | VERIFIED | `create_app()` in `app.py:90-162` accepts 6 optional kwargs; zero uses of `dependency_overrides` |
| LRN-008 | VERIFIED | Real/Recording/Fake executor in `executor.py:61-236` with strict mode; 4 contract test classes |
| LRN-009 | VERIFIED | `register_handler()` on both queue implementations; `make_scan_handler()` factory in `scan.py:53` |
| LRN-011 | VERIFIED | Rust validates path/speed/crf/codec; Python validates business policy (`allowed_roots` in scan service) |
| LRN-012 | VERIFIED | Rust used for FFmpegCommand builder and filter string generation (string-heavy); Python app layer does not call `escape_filter_text` directly |
| LRN-013 | VERIFIED | CI Rust coverage `--fail-under-lines 75` (not yet ratcheted to 90% target); Python `fail_under = 80` |
| LRN-016 | VERIFIED | Clip model has NO effects field in Rust `clip/mod.rs:63-76`, Python `models.py:60-74`, or API schemas -- must be added |
| LRN-019 | VERIFIED | v004: test-foundation first; v005: frontend-foundation first; infrastructure-first ordering confirmed |
| LRN-025 | VERIFIED | 20 handoff documents across v001-v005; v005 handoffs communicated integration points enabling 0 rework |

## Unresolved Questions

- **Font file paths**: The drawtext filter requires a `fontfile` parameter on systems without fontconfig. The available system fonts and default font path are platform-dependent and must be resolved at runtime. Mark as: **TBD - requires runtime font discovery**.
- **boxborderw multi-value syntax**: The `boxborderw=10|20|30|40` pipe-delimited syntax for per-side borders is only available in FFmpeg 5.0+. The minimum supported FFmpeg version for stoat-and-ferret is not yet established. Mark as: **TBD - depends on minimum FFmpeg version policy**.

## Recommendations

1. **Expression engine type hierarchy**: Model as `Expr` enum with variants: `Const(f64)`, `Var(Variable)`, `BinaryOp(Op, Box<Expr>, Box<Expr>)`, `UnaryOp(UnaryOp, Box<Expr>)`, `Func(FuncName, Vec<Expr>)`, `If(Box<Expr>, Box<Expr>, Box<Expr>)`. Serialize via `Display` trait.
2. **Graph validation**: Implement Kahn's algorithm directly in the FilterGraph module rather than adding petgraph as a dependency. The graph is small (typically <20 nodes) and the algorithm is straightforward.
3. **atempo chaining**: Implement as a function that takes a speed factor and returns `Vec<f64>` of atempo values, each within [0.5, 2.0]. Use `log2(speed)` to calculate the number of 2.0x stages needed.
4. **Proptest strategy**: Use `prop_oneof!` for expression enum variants with weighted generation favoring common patterns (between, if, arithmetic) over edge cases (bitwise, trigonometric).
5. **drawtext builder**: Use the existing builder pattern (Filter::new().param()) extended with typed methods for each parameter category. Alpha animation delegates to the expression engine.
