# Frame-Accurate Timeline Math

## Problem

Floating-point arithmetic accumulates rounding errors over time. In video editing, this causes:
- Frames drifting out of sync
- Off-by-one errors at cut points
- Inconsistent behavior across platforms

## Solution

Use integer frame counts internally, with rational frame rates.

### Frame Counts as u64

```rust
pub struct Position {
    frames: u64,  // Exact frame count, no precision loss
}

pub struct Duration {
    frames: u64,
}
```

### Rational Frame Rates

```rust
pub struct FrameRate {
    numerator: u32,    // e.g., 30000 for NTSC
    denominator: u32,  // e.g., 1001 for NTSC
}
```

This allows exact representation of:
- NTSC: 30000/1001 (≈29.97 fps)
- PAL: 25/1
- Film: 24/1
- NTSC film: 24000/1001 (≈23.976 fps)

### Conversion at Boundaries

Only convert to f64 at API boundaries (e.g., for display or external tools):

```rust
impl Position {
    pub fn to_seconds(&self, frame_rate: &FrameRate) -> f64 {
        (self.frames as f64 * frame_rate.denominator as f64) 
            / frame_rate.numerator as f64
    }
}
```

## Benefits

- Zero accumulation error in internal calculations
- Exact frame alignment guaranteed
- Deterministic behavior across platforms
- Simple arithmetic: addition, subtraction just work

## When to Apply

- Timeline position calculations
- Clip start/end points
- Duration arithmetic
- Any frame-based media timing

## Source

Discovered during stoat-and-ferret v001, Theme 02 (timeline-math).
