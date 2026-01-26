Explore the stoat-and-ferret codebase to answer these v002 design questions:

## Questions to Answer

### Rust Codebase (Priority)

1. **Clip module status**: What exists in `rust/stoat_ferret_core/src/clip/`? Is `Clip` type implemented? What is its API? Is it exposed to Python already?

2. **ValidationError structure**: Does the Rust ValidationError have `field`, `message`, `actual`, `expected` attributes? Check the clip/validation.rs and how ValidationError is defined/used.

3. **TimeRange list operations**: Do `find_gaps` and `merge_ranges` functions exist in Rust? Check `rust/stoat_ferret_core/src/timeline/` for these. If they exist, are they exposed to Python?

4. **py_ prefix inventory**: Find ALL methods/properties in the PyO3 bindings that have `py_` prefixes. List them with their location. Why do they exist - PyO3 limitation or naming collision?

5. **Stub generation binary**: What's in `rust/stoat_ferret_core/src/bin/`? Is this for pyo3-stub-gen? How does stub generation currently work?

### Design Documents

6. **Repository pattern scope**: Read `docs/design/01-roadmap.md` M1.4 section carefully. Does it specify one VideoRepository or separate repositories for videos, metadata, thumbnails?

7. **FFmpeg executor boundary**: From roadmap M1.5 and the EXP-002 exploration docs, clarify: Does Python call Rust to build command then Python executes subprocess? Or something else?

8. **Prometheus client**: What library does the roadmap/design docs specify for metrics? Check `docs/design/07-quality-architecture.md` for specifics.

### Dependencies

9. **Current Python dependencies**: Check `pyproject.toml` for what's already installed. Any database or metrics libraries present?

10. **pyo3-stub-gen usage**: How is pyo3-stub-gen currently configured? Check Cargo.toml and any existing stub generation setup.

## Output Requirements

Create findings in comms/outbox/exploration/design-research-gaps/:

### README.md (required)
First paragraph: Executive summary of findings relevant to v002 design.
Then: Table of answers to each question with links to details.

### rust-types.md
- Clip module status and API
- ValidationError structure
- TimeRange list operations (find_gaps, merge_ranges)
- Complete py_ prefix inventory with rationale

### stub-generation.md
- How stubs are currently generated
- What's in src/bin/
- pyo3-stub-gen configuration
- What would be needed for CI verification

### design-clarifications.md
- Repository pattern scope from roadmap
- FFmpeg executor/Rust boundary
- Prometheus client library choice
- Current dependencies in pyproject.toml

## Guidelines
- Under 200 lines per document
- Include actual code snippets showing current implementation
- Be specific about what EXISTS vs what's MISSING
- Note any discrepancies between stubs and actual Rust API

## When Complete
git add comms/outbox/exploration/design-research-gaps/
git commit -m "exploration: design-research-gaps complete"