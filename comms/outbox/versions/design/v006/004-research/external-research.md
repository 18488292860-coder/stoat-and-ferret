# External Research - v006

## 1. FFmpeg Filter Expression Engine

Source: [FFmpeg Utilities](https://ffmpeg.org/ffmpeg-utils.html), [eval.texi](https://github.com/fhunleth/ffmpeg/blob/master/doc/eval.texi)

**Operators:** Unary `+`/`-`, Binary `*`/`/`/`+`/`-`, Exponent `^`, Semicolon `expr1;expr2` (eval both, return expr2). Comparisons via functions only. Logical: `*` = AND, `+` = OR (non-zero = true).

**Functions (40 total):** Arithmetic: `abs`, `mod`, `max`, `min`, `clip`, `pow`, `sqrt`, `sgn`, `exp`, `log`. Rounding: `ceil`, `floor`, `trunc`, `round`. Trig: `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `sinh`, `cosh`, `tanh`. Comparison: `eq`, `gte`, `lte`, `gt`, `lt`. Conditional: `if(c,t,e)`, `ifnot(c,t,e)`, `between(x,min,max)`, `not(x)`. Bitwise: `bitand`, `bitor`. State: `st(idx,expr)`, `ld(idx)`, `random(idx)`. Detection: `isinf`, `isnan`, `squish`, `gauss`.

**Constants:** `PI`, `E`, `PHI`. **Variables:** 10 internal slots (0-9) via `st`/`ld`. Per-filter variables: `t` (time), `n` (frame), `pos`, `w`, `h`.

**Design implication:** Model as `Expr` enum: `Const(f64)`, `Var(Variable)`, `BinaryOp`, `UnaryOp`, `Func(name, args)`, `If(cond, then, else)`. Core functions for v006: `between`, `if`, `lt`, `gt`, `eq`, `gte`, `lte`, `clip`, `abs`, `min`, `max`, `mod`, `not`.

---

## 2. FFmpeg drawtext Filter

Source: [drawtext docs](https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Video/drawtext.html), [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html)

**Text:** `text` (mandatory), `textfile`, `expansion` (none/normal/strftime). **Font:** `fontfile` (required without fontconfig), `fontsize` (default 16, expr-capable), `fontcolor` (default "black"), `fontcolor_expr`, `font`. **Position:** `x`/`y` (expression, default 0). **Shadow:** `shadowx`/`shadowy`/`shadowcolor`. **Box:** `box` (0/1), `boxcolor`, `boxborderw`, `borderw`, `bordercolor`. **Alpha:** `alpha` (0.0-1.0, expr-capable). **Enable:** `enable` expression.

**Position expression variables:** `w`/`W`/`main_w`, `h`/`H`/`main_h`, `tw`/`text_w`, `th`/`text_h`, `lh`/`line_h`, `max_glyph_a`/`ascent`, `max_glyph_d`/`descent`, `max_glyph_h`, `max_glyph_w`, `sar`, `dar`, `n`, `t`.

**Position presets:** Center: `x=(w-text_w)/2:y=(h-text_h)/2`. Bottom: `x=(w-text_w)/2:y=h-text_h-10`. Margin: `x=10:y=10`.

**Escaping:** Four levels apply. Existing `escape_filter_text()` covers levels 2-3. Drawtext additionally needs `%` -> `%%` for text expansion mode.

**Alpha fade pattern (fade-in, hold, fade-out):**
```
alpha='if(lt(t,T1),0,if(lt(t,T1+FI),(t-T1)/FI,if(lt(t,T2-FO),1,if(lt(t,T2),(T2-t)/FO,0))))'
```
Where T1=start, FI=fade-in duration, T2=end, FO=fade-out duration.

---

## 3. FFmpeg Speed Control (setpts / atempo)

Source: [OTTVerse guide](https://ottverse.com/how-to-speed-up-slow-down-video-playback-using-ffmpeg/), [atempo docs](https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Audio/atempo.html)

**setpts formula:** `setpts=(1.0/speed)*PTS`. Examples: 2x = `0.5*PTS`, 0.5x = `2.0*PTS`, 4x = `0.25*PTS`.

**atempo:** Range [0.5, 100.0], default 1.0. Quality degrades above 2.0 (sample skipping). Chain instances within [0.5, 2.0] for quality.

**Chaining formula:** For S > 2.0: `N = floor(log2(S))`, chain N instances of `atempo=2.0` + remainder. For S < 0.5: `N = floor(log2(1/S))`, chain N instances of `atempo=0.5` + remainder.

| Speed | atempo Chain |
|---|---|
| 0.25x | `atempo=0.5,atempo=0.5` |
| 3.0x | `atempo=2.0,atempo=1.5` |
| 4.0x | `atempo=2.0,atempo=2.0` |

---

## 4. Proptest for Rust

Source: [proptest-rs/proptest](https://deepwiki.com/proptest-rs/proptest)

**Manual strategies** (preferred -- `proptest-derive` not in Cargo.toml):
```rust
fn expr_strategy() -> impl Strategy<Value = Expr> {
    prop_oneof![
        any::<f64>().prop_map(Expr::Const),
        (any::<f64>(), any::<f64>()).prop_map(|(a, b)| Expr::Add(Box::new(Expr::Const(a)), Box::new(Expr::Const(b)))),
    ]
}
```

**`prop_compose!`** for constrained values: `fn valid_speed()(speed in 0.25f64..=4.0) -> SpeedControl`.

**Roundtrip pattern:** Generate expr -> serialize -> verify non-empty, no NaN/Infinity. Already used in `timeline/range.rs` (overlap symmetry) and `timeline/mod.rs` (frame roundtrip).

---

## 5. Graph Validation Algorithms

Source: [Kahn's algorithm](https://gaultier.github.io/blog/kahns_algorithm.html), [GeeksforGeeks](https://www.geeksforgeeks.org/dsa/topological-sorting-indegree-based-solution/)

**Kahn's algorithm (recommended):** Compute in-degree per node, enqueue nodes with in-degree 0, process queue (decrement successors, enqueue when in-degree reaches 0). If result < total nodes, cycle exists. O(V+E) time, O(V) space.

**Pad label matching:** Build `HashMap<output_label, chain_index>`. For each input label, look up in map (missing = error). Special-case `N:v`/`N:a` as stream refs. Unmatched outputs are graph final outputs.

**petgraph vs inline:** FilterGraph has <20 chains. Kahn's is ~30 lines. Domain-specific error messages (BL-038 AC #4) favor inline implementation over generic petgraph errors.
