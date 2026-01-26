# Security Validation Whitelist Pattern

## Principle

For security-sensitive inputs, prefer whitelisting known-good values over blacklisting dangerous ones.

## Why Whitelisting Wins

**Blacklist problems:**
- Incomplete: attackers find values you didn't think to block
- Maintenance burden: must update as new attacks emerge
- False sense of security: "we block bad things" vs "we only allow good things"

**Whitelist benefits:**
- Complete by default: unknown values rejected
- Explicit: clear what's allowed
- Future-proof: new attack vectors automatically blocked

## Example: FFmpeg Codec Validation

**Bad (blacklist):**
```rust
fn validate_codec(codec: &str) -> bool {
    let dangerous = ["eval", "shell", "exec"];
    !dangerous.iter().any(|d| codec.contains(d))
}
```

**Good (whitelist):**
```rust
const ALLOWED_VIDEO_CODECS: &[&str] = &[
    "libx264", "libx265", "libvpx", "libvpx-vp9",
    "libaom-av1", "copy", "h264", "hevc",
];

fn validate_codec(codec: &str) -> Result<(), ValidationError> {
    if ALLOWED_VIDEO_CODECS.contains(&codec) {
        Ok(())
    } else {
        Err(ValidationError::new("codec", "not in allowed list"))
    }
}
```

## Application Areas

- Codec names for FFmpeg
- File path extensions
- Filter names
- Preset names
- Any user-provided value that maps to system behavior

## Trade-offs

- Whitelist requires explicit additions for new valid values
- May need configuration to extend the list
- Document how to add new values safely

## Source

Discovered during stoat-and-ferret v001, Theme 03 (command-builder, input-sanitization).
