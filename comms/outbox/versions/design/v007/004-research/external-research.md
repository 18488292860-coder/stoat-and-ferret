# External Research - v007

## FFmpeg Audio Filters (BL-044)

### amix Filter
Mixes multiple audio inputs into a single output. Only supports float samples.

**Parameters** (source: [FFmpeg Filters Docs](https://ffmpeg.org/ffmpeg-filters.html)):
- `inputs`: Number of inputs (default: 2)
- `duration`: `longest` (default), `shortest`, `first`
- `dropout_transition`: Seconds for volume renormalization when input ends (default: 2)
- `weights`: Per-input weights as space-separated numbers (default: 1 per input)
- `normalize`: Scale inputs (default: enabled). Disable with caution — heavy clipping risk

**Syntax**: `[0:a][1:a]amix=inputs=2:duration=first:weights=1 0.25`

### volume Filter
Adjusts audio volume by multiplication factor or dB.

**Parameters**:
- `volume`: Expression or value (default: "1.0"). Accepts linear (0.5 = half) or dB ("3dB")
- `precision`: `fixed` (U8/S16/S32), `float` (FLT, default), `double` (DBL)
- `replaygain`: `drop` (default), `ignore`, `track`, `album`

**Syntax**: `volume=0.5` or `volume=3dB`

### afade Filter
Audio fade in/out with configurable curves.

**Parameters**:
- `type` (t): `in` (default) or `out`
- `start_time` (st): Fade start in seconds (default: 0)
- `duration` (d): Fade duration in seconds
- `curve` (c): 12 available curves — `tri` (linear, default), `qsin`, `hsin`, `esin`, `log`, `ipar`, `qua`, `cub`, `squ`, `cbr`, `par`

**Syntax**: `afade=t=in:st=0:d=3:c=qsin`

### sidechaincompress Filter (Audio Ducking)
Compresses first input based on second input signal level. Two inputs, one output.

**Parameters** (all support runtime commands):
- `level_in`: Input gain (default: 1, range: 0.015625-64)
- `threshold`: Level above which compression starts (default: 0.125, range: 0.00097563-1)
- `ratio`: Compression ratio (default: 2, range: 1-20)
- `attack`: Rise time in ms before compression (default: 20, range: 0.01-2000)
- `release`: Fall time in ms before decompression (default: 250, range: 0.01-9000)
- `makeup`: Post-compression amplification (default: 1, range: 1-64)
- `knee`: Soft knee curve (default: 2.82843, range: 1-8)
- `link`: `average` (default) or `maximum` channel level
- `detection`: `rms` (default, smoother) or `peak` (exact)
- `level_sc`: Sidechain gain (default: 1, range: 0.015625-64)
- `mix`: Wet/dry mix (default: 1, range: 0-1)

**Ducking pattern**: `[speech]asplit[sc][mix];[music][sc]sidechaincompress=threshold=0.1:ratio=4:attack=10:release=500[duck];[duck][mix]amerge`

## FFmpeg Video Transition Filters (BL-045)

### fade Filter (Video)
Fade video to/from a solid color.

**Parameters**:
- `type` (t): `in` or `out` (default: in)
- `start_time` (st): Start in seconds (default: 0)
- `duration` (d): Duration in seconds
- `nb_frames` (n): Alternative to duration (default: 25 frames)
- `alpha`: Fade only alpha channel if 1 (default: 0)
- `color` (c): Fade color (default: "black")

**Syntax**: `fade=t=in:st=0:d=3:c=black`

### xfade Filter
Cross-fade between two video inputs. Both must have same resolution, frame rate, and pixel format.

**Parameters**:
- `transition`: 64 available types (default: `fade`):
  - Basic: `fade`, `fadeblack`, `fadewhite`, `fadegrays`, `fadefast`, `fadeslow`
  - Wipes: `wipeleft`, `wiperight`, `wipeup`, `wipedown`, `wipetl`, `wipetr`, `wipebl`, `wipebr`
  - Slides: `slideleft`, `slideright`, `slideup`, `slidedown`
  - Smooth: `smoothleft`, `smoothright`, `smoothup`, `smoothdown`
  - Shapes: `circlecrop`, `rectcrop`, `circleopen`, `circleclose`, `radial`
  - Bars: `vertopen`, `vertclose`, `horzopen`, `horzclose`
  - Effects: `dissolve`, `pixelize`, `distance`, `hblur`
  - Diagonal: `diagtl`, `diagtr`, `diagbl`, `diagbr`
  - Slices: `hlslice`, `hrslice`, `vuslice`, `vdslice`
  - Squeeze: `squeezeh`, `squeezev`
  - Zoom: `zoomin`
  - Wind: `hlwind`, `hrwind`, `vuwind`, `vdwind`
  - Cover/Reveal: `coverleft`, `coverright`, `coverup`, `coverdown`, `revealleft`, `revealright`, `revealup`, `revealdown`
  - Custom: `custom` (with expr parameter)
- `duration`: 0-60 seconds (default: 1)
- `offset`: Start relative to first input in seconds (default: 0)
- `expr`: Custom transition expression (variables: X, Y, W, H, P, PLANE, A, B)

**Syntax**: `[0:v][1:v]xfade=transition=wipeleft:duration=2:offset=5[out]`

### acrossfade Filter (Audio)
Cross-fade between audio inputs near the end of first stream.

**Parameters**:
- `inputs` (n): Number of inputs (default: 2)
- `duration` (d): Cross-fade duration in seconds
- `nb_samples` (ns): Alternative to duration (default: 44100)
- `overlap` (o): Overlap first end with second start (default: enabled)
- `curve1`, `curve2`: Fade curves (same 12 types as afade)

**Syntax**: `[0:a][1:a]acrossfade=d=2:c1=tri:c2=tri`

## React JSON Schema Form Generation (BL-049)

### Recommended: @rjsf/core (React JSON Schema Form)
Source: [rjsf-team/react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form), DeepWiki analysis

**Why RJSF**:
- Mature, actively maintained, 14k+ GitHub stars
- Built-in JSON Schema validation via `@rjsf/validator-ajv8`
- Extensive widget support matching BL-049 requirements
- uiSchema mechanism for rendering control without modifying data schema
- Headless / theme-agnostic (works with any UI library)

**Built-in widgets matching BL-049 needs**:
| BL-049 Requirement | RJSF Widget | uiSchema Config |
|---|---|---|
| Number with range slider | `RangeWidget` | `"ui:widget": "range"` |
| String input | Default text input | (none needed) |
| Enum dropdown | Default select | (auto from `enum` in schema) |
| Boolean toggle | Default checkbox | (none needed) |
| Color picker | `ColorWidget` | `"ui:widget": "color"` |

**Custom widget support**: Register custom components via `widgets` prop. Custom widgets receive `id`, `value`, `onChange`, `schema`, `uiSchema` props.

**uiSchema mechanism**: Controls how fields render without changing the data schema. Key properties: `ui:widget`, `ui:options`, `ui:classNames`, `ui:placeholder`, `ui:readonly`.

**Alternative considered: JSON Forms** ([jsonforms.io](https://jsonforms.io)): Also schema-driven, but heavier and more opinionated on layout. RJSF is simpler and more flexible for custom UI.

**Approach for v007**: Do NOT add RJSF as a dependency. Build a lightweight custom form generator that reads the existing `parameter_schema` format (already JSON Schema compatible). This avoids a heavy dependency for what is a relatively simple form generation need (5-6 parameter types). If complexity grows, migrate to RJSF later. Use RJSF's widget mapping as a design reference.

## FFmpeg Syntax Highlighting (BL-050)

No dedicated FFmpeg syntax highlighting library exists. Options:
1. **Custom tokenizer with Prism.js**: Define FFmpeg filter grammar (filter names, params, pad labels)
2. **Plain monospace with keyword coloring**: Simple regex-based highlighting for filter names and `[pad]` labels
3. **Code block with generic highlighting**: Use a `<pre><code>` block with no language-specific highlighting

**Recommendation**: Option 2 — simple regex coloring is sufficient for v007. Filter strings are short and the structure is `name=key=value:key=value`. Color filter names and bracket-wrapped pad labels.

## Preview Thumbnails (BL-051)

Frame extraction command: `ffmpeg -ss {timestamp} -i {input} -frames:v 1 -vf {filter_string} thumb.png`

**For v007**: Preview thumbnails require server-side FFmpeg execution which adds significant complexity (temp files, cleanup, latency). Recommend deferring actual thumbnail preview to a future version. Instead, use the filter string preview (BL-050) as the primary transparency mechanism for v007. The existing FFmpegCommand builder (command.rs) can compose the extraction command when needed.
