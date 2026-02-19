# Discrepancies - v007

## Blocking Issues

None.

## Non-Blocking Observations

### 1. Trailing Newline in Persisted Files

**Severity:** Cosmetic
**Scope:** All 22 feature documents (requirements.md + implementation-plan.md)
**Detail:** Draft files end with a trailing newline (`\n` at EOF) per POSIX convention. Persisted files are missing the final newline. This is caused by the persistence mechanism stripping the trailing newline during file write.
**Impact:** None. Content is identical. No execution impact.

### 2. BL-051 AC #3 — Preview Thumbnail Scope Change

**Severity:** Informational (intentional design decision)
**Detail:** BL-051 acceptance criterion #3 specifies "Preview thumbnail displays a static frame with the effect applied." The feature requirements implement this as a filter string preview panel instead of a rendered thumbnail. The change is documented in the requirements.md design clarification section with rationale: "This satisfies the transparency intent. Rendered thumbnails documented as future enhancement."
**Impact:** Execution proceeds with filter string preview. This is not a gap — it's a deliberate scope decision made during design.

### 3. T03 THEME_DESIGN.md — Missing Dependents

**Severity:** Low
**Detail:** T03's THEME_DESIGN.md does not explicitly list T04 as a dependent. T04's THEME_DESIGN.md correctly states it depends on T03. The dependency chain is complete from the consumer side.
**Impact:** None for execution. Could improve bidirectional documentation.

### 4. Coverage Threshold Wording

**Severity:** Low
**Detail:** The risk assessment specifies ">90% coverage for new Rust modules." Some implementation plan quality gates sections reference "80%" as the threshold (matching the project's overall minimum from AGENTS.md). The feature requirements.md files correctly specify ">=90%" where applicable.
**Impact:** None. The authoritative threshold is in the requirements.md files which the executor reads.
