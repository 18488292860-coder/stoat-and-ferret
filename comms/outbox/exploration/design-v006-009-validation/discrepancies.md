# Discrepancies - v006 Pre-Execution Validation

## Non-Blocking Warnings

### 1. THEME_INDEX.md Feature Description Placeholders

**Severity**: Low (cosmetic)
**Location**: `comms/inbox/versions/execution/v006/THEME_INDEX.md`

The persisted THEME_INDEX.md uses `_Feature description_` placeholders for all 8 features instead of the actual descriptions from the Task 007 drafts. For example:

- Draft: `- 001-expression-engine: Implement type-safe FFmpeg expression AST with builder API, serialization, property-based tests, and PyO3 bindings`
- Persisted: `- 001-expression-engine: _Feature description_`

**Impact**: Minimal. Feature descriptions exist in full in each feature's `requirements.md` and in the THEME_DESIGN.md feature tables. The THEME_INDEX serves as a navigation aid; implementers will read the feature-level documents regardless.

**Root cause**: The `design_version` tool generates THEME_INDEX with placeholder descriptions as a template. The persist step (Task 008) copied the tool-generated version rather than the draft version.

### 2. VERSION_DESIGN.md Reformatting

**Severity**: Low
**Location**: `comms/inbox/versions/execution/v006/VERSION_DESIGN.md`

The persisted VERSION_DESIGN.md was reformatted by the `design_version` tool, which:
- Dropped the "Key Design Decisions" section (5 decisions: expression whitelist, clip effects storage, effect registry pattern, contract tests, speed control approach)
- Dropped the "Design Artifacts" navigation section (links to 6 analysis folders)
- Added boilerplate sections (Success Criteria checkboxes, Backlog Items with BACKLOG.md links)
- Added constraints not in the draft (py_ prefix convention, asyncio.TimeoutError, Python 80% coverage)

**Impact**: Minimal. The 5 design decisions are documented in the feature-level requirements and implementation plans. The design artifact links are in the THEME_DESIGN files. The added constraints are correct (from AGENTS.md and project conventions).

### 3. Missing `rust/stoat_ferret_core/tests/` Directory

**Severity**: Low (implementation-time)
**Location**: 5 implementation plans reference creating integration test files in `rust/stoat_ferret_core/tests/`

The existing Rust crate uses inline `#[cfg(test)]` modules within source files (12+ files use this pattern). The `tests/` directory for Rust integration tests does not exist.

**Impact**: Non-blocking. Cargo natively supports both inline tests and integration tests in `tests/`. Implementers can either:
1. Create the `tests/` directory when implementing the first feature (expression-engine)
2. Use inline `#[cfg(test)] mod tests { }` within source files (matching existing convention)

Both approaches produce identical test behavior. The implementation plans describe the test content correctly regardless of file organization.

## Blocking Issues

No discrepancies identified that would block execution.
