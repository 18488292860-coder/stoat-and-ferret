## Exploration: pydantic-settings Current Patterns

### Context
stoat-and-ferret v003 needs externalized configuration for FastAPI. pydantic-settings v2 has different APIs from v1. We need current best practices.

### Questions to Answer

1. **Current API**: What's the pydantic-settings v2 API for defining settings classes? BaseSettings location, model_config vs class Config?

2. **Environment variable handling**: How are env vars mapped to fields? Prefixes? Nested settings?

3. **Validation patterns**: How do we validate settings (e.g., database path exists, port in range)?

4. **Testing patterns**: How do we override settings in tests? Environment manipulation vs direct instantiation?

5. **FastAPI integration**: What's the recommended pattern for settings as FastAPI dependencies?

### Research Sources
- DeepWiki: pydantic/pydantic-settings
- Web search for pydantic-settings v2 migration guide if needed

### Output Requirements

Write findings to: `comms/outbox/exploration/pydantic-settings-current/`

Required files:
- `README.md` - Summary with recommended patterns
- `settings-class.md` - Concrete Settings class example for stoat-and-ferret
- `validation-patterns.md` - Custom validators and field types
- `testing-patterns.md` - How to override in tests
- `fastapi-integration.md` - Dependency injection pattern

### Commit Instructions
Commit all files with message: "docs(exploration): pydantic-settings current patterns for v003"