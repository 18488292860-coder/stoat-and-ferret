# Evidence Log - v007

## Volume Range (BL-044)

- **Value**: 0.0-10.0 (linear scale)
- **Source**: `rust/stoat_ferret_core/src/sanitize/mod.rs:319-356` — `validate_volume()`
- **Data**: Existing Rust validation function already enforces this range
- **Rationale**: 10.0 allows 10x amplification. Matches existing validator. FFmpeg volume filter accepts any positive float but extreme values cause clipping. 10x is a reasonable upper bound.

## Speed Factor Range (BL-044 audio)

- **Value**: 0.25-4.0
- **Source**: `rust/stoat_ferret_core/src/sanitize/mod.rs:280-317` — `validate_speed()`
- **Data**: Existing SpeedControl already validates this range. atempo filter quality range is [0.5, 2.0], chained for wider ranges.
- **Rationale**: Proven in v006. Audio tempo chaining handles 0.25-4.0 correctly.

## amix Input Count Range

- **Value**: 2-32 inputs
- **Source**: FFmpeg docs — amix defaults to 2 inputs; practical limit ~32 before audio quality degrades
- **Data**: Default is 2. No hard maximum in FFmpeg but quality and CPU usage scale linearly.
- **Rationale**: 32 is a generous upper bound for video editing. Most use cases need 2-4 tracks.

## afade Curve Types

- **Value**: 12 curves — tri, qsin, hsin, esin, log, ipar, qua, cub, squ, cbr, par
- **Source**: FFmpeg docs section 8.22 afade
- **Data**: `tri` (linear) is default. Expression engine (Expr) could generate custom curves but these built-in curves cover common needs.
- **Rationale**: Map these to a Rust enum. Default to `tri` for simplicity.

## xfade Transition Types

- **Value**: 64 types (see external-research.md for full list)
- **Source**: FFmpeg docs section 11.285 xfade
- **Data**: Default is `fade`. All types are string identifiers.
- **Rationale**: Map to a Rust enum for type safety and validation. The enum approach matches the Position preset pattern in DrawtextBuilder.

## xfade Duration Range

- **Value**: 0.0-60.0 seconds
- **Source**: FFmpeg docs section 11.285 xfade — "Range is 0 to 60 seconds"
- **Data**: Default is 1.0 second. Values above 60 are rejected by FFmpeg.
- **Rationale**: Use FFmpeg's own range. Most transitions are 0.5-3.0 seconds.

## Video Fade Color

- **Value**: Any FFmpeg color string (default: "black")
- **Source**: FFmpeg docs section 11.86 fade
- **Data**: Accepts named colors (black, white, red) and hex (#RRGGBB)
- **Rationale**: Validate against a reasonable set of named colors or accept any string (FFmpeg validates at execution time).

## sidechaincompress Parameters (Audio Ducking)

- **Threshold**: 0.00097563-1.0 (default: 0.125)
- **Ratio**: 1-20 (default: 2)
- **Attack**: 0.01-2000 ms (default: 20)
- **Release**: 0.01-9000 ms (default: 250)
- **Source**: FFmpeg docs section 8.105 sidechaincompress
- **Data**: All parameters support runtime commands for live adjustment
- **Rationale**: Use FFmpeg defaults as builder defaults. These are well-established compressor settings.

## Debounce Interval for Preview API

- **Value**: 300ms (initial)
- **Source**: `gui/src/hooks/useDebounce.ts:1-12` — existing debounce default
- **Data**: Current useDebounce hook uses 300ms for search queries. Preview API latency unknown pre-implementation.
- **Rationale**: 300ms is standard for type-ahead debouncing. Matches existing codebase convention. Adjust based on measured API latency post-implementation.

## Prometheus Counter Name

- **Value**: `stoat_ferret_effect_applications_total`
- **Source**: Naming convention from `src/stoat_ferret/ffmpeg/metrics.py:11` — `stoat_ferret_ffmpeg_executions_total`
- **Data**: Prefix `stoat_ferret_` matches existing metrics namespace. Label: `effect_type`.
- **Rationale**: Consistent naming with existing metrics. Counter type is correct (monotonically increasing).

## Effect Storage Index-Based CRUD

- **Value**: Array index (0-based) for effect identification
- **Source**: `src/stoat_ferret/api/routers/effects.py:218-225` — effects stored as list[dict]
- **Data**: Current storage is append-only JSON array. No unique ID per effect entry.
- **Rationale**: Array index is sufficient for CRUD within a clip's effect list. Adding UUIDs per effect would require schema migration and adds complexity without clear benefit for v007 scope.
