# Evidence Log - v006

## atempo Valid Range

- **Value:** [0.5, 100.0]
- **Source:** [FFmpeg atempo 7.1.1 documentation](https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Audio/atempo.html), [FFmpeg source af_atempo.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/af_atempo.c)
- **Data:** Official docs state "Tempo must be in the [0.5, 100.0] range." The source code defines `TEMPO_MIN 0.5` and `TEMPO_MAX 100.0`.
- **Rationale:** Quality degrades above 2.0 (samples skipped instead of blended). For the v006 speed builder, each chained instance should stay within [0.5, 2.0] to maintain audio quality.

## atempo Quality Threshold

- **Value:** 2.0
- **Source:** [FFmpeg atempo documentation](https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Audio/atempo.html)
- **Data:** "Tempo greater than 2 will skip some samples rather than blend them in. If for any reason this is a concern it is always possible to daisy-chain several instances of atempo to achieve the desired product tempo."
- **Rationale:** The builder should chain atempo instances, each within [0.5, 2.0], rather than using a single high value.

## Speed Range for v006

- **Value:** [0.25, 4.0]
- **Source:** Existing `validate_speed()` in `rust/stoat_ferret_core/src/sanitize/mod.rs` line 306
- **Data:** `if (0.25..=4.0).contains(&speed)` -- already validated in codebase
- **Rationale:** Matches BL-041 AC #1 requirement. 0.25x = 4 atempo instances of 0.5; 4.0x = 2 atempo instances of 2.0.

## setpts Formula

- **Value:** `PTS * (1.0 / speed_factor)`
- **Source:** [OTTVerse setpts guide](https://ottverse.com/how-to-speed-up-slow-down-video-playback-using-ffmpeg/), [FFmpeg setpts.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/setpts.c)
- **Data:** `setpts=0.5*PTS` = 2x speed, `setpts=2.0*PTS` = 0.5x speed. General: `setpts=(1/speed)*PTS`.
- **Rationale:** The speed builder generates the setpts expression by computing `1.0/speed` and formatting as `{factor}*PTS`.

## drawtext Default Font Size

- **Value:** 16
- **Source:** [FFmpeg drawtext 7.1.1 docs](https://ayosec.github.io/ffmpeg-filters-docs/7.1/Filters/Video/drawtext.html)
- **Data:** "The default value of fontsize is 16."
- **Rationale:** The drawtext builder should use 16 as default if no fontsize specified.

## drawtext Alpha Range

- **Value:** [0.0, 1.0]
- **Source:** [FFmpeg drawtext documentation](https://ffmpegbyexample.com/examples/50gowmkq/fade_in_and_out_text_using_the_drawtext_filter/)
- **Data:** "The alpha parameter draws the text applying alpha blending. The value can be either a number between 0.0 and 1.0."
- **Rationale:** Alpha expressions must evaluate to values in [0.0, 1.0]. The expression builder should provide a `clip(expr, 0, 1)` wrapper for safety.

## FFmpeg Expression Function Count

- **Value:** 35+ functions
- **Source:** [FFmpeg Utilities Documentation](https://ffmpeg.org/ffmpeg-utils.html), [eval.texi](https://github.com/fhunleth/ffmpeg/blob/master/doc/eval.texi)
- **Data:** Arithmetic (8), rounding (4), trigonometric (10), comparison (5), conditional (4), bitwise (2), state (3), detection (4) = 40 total functions.
- **Rationale:** The expression engine need not model all 40 functions. Core set for effects: `between`, `if`, `lt`, `gt`, `eq`, `gte`, `lte`, `clip`, `abs`, `min`, `max`, `mod`, `not`. Extended set can be added later.

## FFmpeg Expression Internal Variables

- **Value:** 10 slots (indices 0-9)
- **Source:** [FFmpeg Utilities Documentation](https://ffmpeg.org/ffmpeg-utils.html)
- **Data:** "They can be accessed using the ld and st functions with an index argument varying from 0 to 9."
- **Rationale:** The `st`/`ld` pattern is rarely needed for typical effects. Model as function calls (`Func("st", [Const(idx), expr])`) rather than special syntax.

## proptest Dependency Version

- **Value:** proptest = "1.0"
- **Source:** `rust/stoat_ferret_core/Cargo.toml` line 15
- **Data:** Already in dev-dependencies. No proptest-derive.
- **Rationale:** Use manual strategies (`prop_oneof!`, `prop_compose!`) rather than derive macro.

## Kahn's Algorithm Complexity

- **Value:** O(V + E) time, O(V) space
- **Source:** [Wikipedia - Topological sorting](https://en.wikipedia.org/wiki/Topological_sorting), [GeeksforGeeks](https://www.geeksforgeeks.org/dsa/topological-sorting-indegree-based-solution/)
- **Data:** Linear time in graph size. For typical FilterGraphs (V < 20, E < 40), execution is negligible.
- **Rationale:** No need for petgraph dependency. Inline implementation is sufficient.
