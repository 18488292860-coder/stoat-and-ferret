## Context

When writing requirements or specifications that include numeric claims about external systems (e.g., "64 FFmpeg filter types", "12 supported codecs", "N API endpoints").

## Insight

Cross-check numeric claims in requirements against upstream source documentation or code during the design phase, not during implementation. Discrepancies between requirements and reality create confusion during implementation and force deviation documentation. A 5-minute verification during design saves implementation-time surprises and spec deviation debates.

## Evidence

In v007 Theme 01 Feature 002 (transition filter builders), the requirements specified "64 FFmpeg xfade variants" based on early research. During implementation, the actual FFmpeg source code (`libavfilter/vf_xfade.c`) contained 59 transition types (58 built-in + custom). The implementer had to decide whether to pad the enum to match the spec or implement the actual count, then document the deviation in the completion report.

## Application

- When requirements reference counts or capabilities of external tools/libraries, verify against the authoritative source (source code, official docs)
- Include the verification source in the requirements document (e.g., "59 variants per FFmpeg libavfilter/vf_xfade.c")
- Flag numeric claims as "approximate" or "verified" in the design document
- Prefer linking to upstream source rather than hardcoding counts that may change