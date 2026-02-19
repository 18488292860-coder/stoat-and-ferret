# Exploration: design-v007-004-research

Research and investigation for v007 version design, covering all 9 backlog items (BL-044 through BL-052).

## What Was Produced

5 research documents saved to `comms/outbox/versions/design/v007/004-research/`:

1. **README.md** — Research summary with questions, findings, unresolved items, and recommendations
2. **codebase-patterns.md** — Existing Rust builder, Python registry, DI, effects storage, and GUI patterns with file paths and line references. Includes learning verification table (12 learnings verified)
3. **external-research.md** — FFmpeg filter documentation (amix, volume, afade, sidechaincompress, fade, xfade, acrossfade), RJSF form library analysis, syntax highlighting options, preview thumbnail approach
4. **evidence-log.md** — Concrete parameter values with sources: volume range (0.0-10.0), speed range (0.25-4.0), xfade transition types (64), afade curves (12), sidechaincompress ranges, debounce interval (300ms)
5. **impact-analysis.md** — Dependencies across Rust/Python/GUI layers, breaking changes (none external), test infrastructure needs, documentation update requirements

## Key Findings

- All 12 referenced learnings (LRN-001 through LRN-031) verified as still applicable
- FFmpeg provides complete audio filter coverage (amix, volume, afade, sidechaincompress for ducking)
- 64 xfade transition types available, maps well to Rust enum pattern
- Existing builder pattern extends cleanly to new filter types
- Registry refactoring has clear path: add `build_fn` to EffectDefinition
- Custom form generator recommended over RJSF dependency for v007 scope
- Preview thumbnails deferred to future version; filter string preview is sufficient
