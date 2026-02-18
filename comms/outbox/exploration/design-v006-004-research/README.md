# Exploration: design-v006-004-research

Research and investigation for v006 effects engine design covering FFmpeg filter expressions, drawtext parameters, atempo chaining, Rust proptest patterns, and graph validation algorithms.

## Artifacts Produced

All outputs saved to `comms/outbox/versions/design/v006/004-research/`:

| File | Description |
|---|---|
| `README.md` | Research summary, findings, unresolved questions, recommendations |
| `external-research.md` | FFmpeg expression grammar, drawtext parameters, atempo chaining, proptest strategies, graph algorithms |
| `codebase-patterns.md` | Existing filter system, PyO3 builder pattern, proptest usage, module organization |
| `evidence-log.md` | Concrete values with sources: atempo range, setpts formula, defaults, complexity bounds |
| `impact-analysis.md` | Dependencies, breaking changes, test needs, documentation updates |

## Key Findings

1. FFmpeg expression engine has 40+ functions mappable to a Rust enum hierarchy
2. drawtext has 20+ parameters; alpha expressions enable fade animations
3. atempo range is [0.5, 100.0] but quality-safe range is [0.5, 2.0] per instance
4. proptest already in Cargo.toml; manual strategies via prop_oneof!/prop_compose! preferred
5. Kahn's algorithm recommended over petgraph dependency for graph validation

## Sources Consulted

- FFmpeg official documentation (ffmpeg.org/ffmpeg-utils.html, ffmpeg.org/ffmpeg-filters.html)
- FFmpeg filter docs mirror (ayosec.github.io/ffmpeg-filters-docs/7.1/)
- proptest-rs/proptest via DeepWiki
- Multiple web sources for algorithms and examples (cited in external-research.md)
