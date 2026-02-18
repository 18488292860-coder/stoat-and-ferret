# Draft Checklist — v006 Document Drafts

- [x] manifest.json is valid JSON with all required fields
- [x] Every theme in manifest has a corresponding folder under drafts/
- [x] Every feature in manifest has a corresponding folder with both requirements.md and implementation-plan.md
- [x] VERSION_DESIGN.md and THEME_INDEX.md exist in drafts/
- [x] THEME_INDEX.md feature lines match format `- \d{3}-[\w-]+: .+`
- [x] No placeholder text in any draft (`_Theme goal_`, `_Feature description_`, `[FILL IN]`, `TODO`)
- [x] All backlog IDs from manifest appear in at least one requirements.md
- [x] No theme or feature slug starts with a digit prefix (`^\d+-`)
- [x] Backlog IDs in each requirements.md cross-referenced against Task 002 backlog analysis (no mismatches)
- [x] All "Files to Modify" paths verified via Glob structure queries (no unverified paths)

## Backlog ID Cross-Reference

| Backlog ID | Task 002 Title | requirements.md Location | Match |
|-----------|---------------|------------------------|-------|
| BL-037 | Implement FFmpeg filter expression engine in Rust | filter-engine/expression-engine/requirements.md | Yes |
| BL-038 | Implement filter graph validation for pad matching | filter-engine/graph-validation/requirements.md | Yes |
| BL-039 | Build filter composition system for chaining, branching, and merging | filter-engine/filter-composition/requirements.md | Yes |
| BL-040 | Implement drawtext filter builder for text overlays | filter-builders/drawtext-builder/requirements.md | Yes |
| BL-041 | Implement speed control filter builders (setpts/atempo) | filter-builders/speed-builders/requirements.md | Yes |
| BL-042 | Create effect discovery API endpoint | effects-api/effect-discovery/requirements.md | Yes |
| BL-043 | Create API endpoint to apply text overlay effect to clips | effects-api/clip-effect-api/requirements.md | Yes |

## Slug Verification

| Type | Slug | Starts with digit? |
|------|------|-------------------|
| Theme | filter-engine | No |
| Theme | filter-builders | No |
| Theme | effects-api | No |
| Feature | expression-engine | No |
| Feature | graph-validation | No |
| Feature | filter-composition | No |
| Feature | drawtext-builder | No |
| Feature | speed-builders | No |
| Feature | effect-discovery | No |
| Feature | clip-effect-api | No |
| Feature | architecture-docs | No |

## THEME_INDEX.md Format Verification

Each feature line matches `- \d{3}-[\w-]+: .+`:
- `- 001-expression-engine: Type-safe Rust expression builder...` ✓
- `- 002-graph-validation: Validate FilterGraph pad matching...` ✓
- `- 003-filter-composition: Programmatic chain, branch, and merge...` ✓
- `- 001-drawtext-builder: Type-safe drawtext filter builder...` ✓
- `- 002-speed-builders: setpts and atempo filter builders...` ✓
- `- 001-effect-discovery: Effect registry with parameter schemas...` ✓
- `- 002-clip-effect-api: POST endpoint to apply text overlay...` ✓
- `- 003-architecture-docs: Update 02-architecture.md...` ✓
