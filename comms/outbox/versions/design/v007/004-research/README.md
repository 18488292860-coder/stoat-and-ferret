# 004 Research and Investigation - v007

Research covered all 9 backlog items (BL-044 through BL-052) for v007's audio mixing, transitions, effect registry, GUI effect workshop, and E2E testing scope. Key findings: FFmpeg provides all needed audio/transition filters with well-documented parameters; the existing builder pattern in Rust extends naturally to new filter types; RJSF is a strong fit for JSON schema-driven form generation; and the if/elif dispatch refactoring has a clear registry-based replacement path. All 12 referenced learnings were verified against the codebase.

## Research Questions

### BL-044: Audio Mixing Filter Builders
1. What FFmpeg audio filters are needed? (amix, volume, afade, sidechaincompress)
2. What are the valid parameter ranges for each filter?
3. How does audio ducking work in FFmpeg?
4. Does the existing expression engine support audio fade curves?

### BL-045: Transition Filter Builders
5. What are all available xfade transition types?
6. What parameters do fade (video), xfade, and acrossfade accept?
7. How do two-input filters differ from single-input in the builder pattern?

### BL-047: Effect Registry
8. How should `_build_filter_string()` be refactored to registry dispatch?
9. What JSON Schema subset is needed for form generation?
10. How should the builder protocol work for DI?

### BL-049: Dynamic Parameter Forms
11. What library should generate forms from JSON Schema?
12. What widget types are needed (slider, color picker, dropdown)?

### BL-050: Live Filter Preview
13. How should preview API calls be debounced?
14. Is FFmpeg syntax highlighting available as a library?

### BL-051: Effect Builder Workflow
15. How do preview thumbnails work (frame extraction + effect)?
16. What API changes are needed for effect CRUD?

## Findings Summary

- **FFmpeg filters fully documented**: amix (inputs, duration, weights, normalize), volume (linear/dB), afade (type, duration, curve with 12 types), sidechaincompress (threshold, ratio, attack, release), xfade (64 transition types, duration, offset), acrossfade (duration, curves), video fade (type, duration, color)
- **Builder pattern extends cleanly**: Audio/transition builders follow identical Rust builder -> PyO3 -> stubs -> tests template from v006 (LRN-028 verified)
- **Two-input filters are new**: xfade and sidechaincompress need two input streams. FilterChain already supports multiple inputs via `.input()` calls
- **Registry refactoring path clear**: Replace `_build_filter_string()` if/elif with `build_fn` callable on EffectDefinition. Each effect type registers its own builder function
- **RJSF is the recommended form library**: Built-in range, color, select widgets; uiSchema controls rendering; AJV8 validation matches JSON Schema
- **No FFmpeg-specific syntax highlighting**: Use generic code highlighting (Prism.js or similar) with custom tokenizer or plain monospace display
- **Preview thumbnails**: `ffmpeg -ss {time} -i {input} -frames:v 1 -vf {filters} thumb.png` — compose via FFmpegCommand builder

## Unresolved Questions

1. **Debounce interval for live preview**: TBD - requires runtime testing. Start with 300ms (matches existing `useDebounce` hook default), adjust based on API latency
2. **Preview thumbnail resolution**: TBD - depends on UI layout. Suggest 320x180 as initial target
3. **Effect ordering storage**: Current `effects_json` is a JSON array (ordered). Ordering is implicit by array index. Explicit `order` field may be needed if reordering is required

## Recommendations

1. **Audio builders first**: BL-044 before BL-045 — amix/volume/afade are simpler, establishes template for transitions
2. **Registry as refactoring**: BL-047 should add `build_fn: Callable` to EffectDefinition, replacing `_build_filter_string()` dispatch
3. **RJSF for forms**: Use `@rjsf/core` with custom uiSchema derived from effect parameter_schema
4. **Simple preview**: Debounced API call returning filter string (no server-side rendering needed for v007)
5. **Effect CRUD via existing pattern**: PATCH/DELETE endpoints on `/projects/{id}/clips/{id}/effects/{index}` using array index

## Artifacts

| File | Description |
|------|-------------|
| [codebase-patterns.md](./codebase-patterns.md) | Existing code patterns and extension points |
| [external-research.md](./external-research.md) | FFmpeg filter docs, RJSF analysis, library evaluation |
| [evidence-log.md](./evidence-log.md) | Concrete parameter values with sources |
| [impact-analysis.md](./impact-analysis.md) | Dependencies, breaking changes, test/doc impacts |
