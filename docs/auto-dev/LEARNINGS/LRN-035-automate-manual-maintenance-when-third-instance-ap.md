## Context

When a manual maintenance task (stub generation, config syncing, schema updates) that was acceptable for 1-2 instances is about to grow to 3+ instances.

## Insight

Apply the "rule of three" to manual maintenance tasks: manual is fine for 1-2 instances, but automate when a third appears. Each new manual instance increases the risk of inconsistency and the maintenance burden. Set an explicit threshold during the first manual instance so the automation trigger is pre-decided, not discovered through accumulated pain.

## Evidence

In v007 Theme 01, `pyo3_stub_gen` does not support enum types, requiring manual stub entries in two separate `.pyi` files for each enum. FadeCurve (v006) and TransitionType (v007) both needed hand-maintained entries in both `src/stoat_ferret_core/_core.pyi` and `stubs/stoat_ferret_core/_core.pyi`. The Theme 01 retrospective explicitly recommended: "If a third enum type is needed, invest in automating enum stub generation." The v007 version retrospective listed this as medium-priority technical debt.

## Application

- When introducing the first manual maintenance instance, document the automation trigger threshold (usually 3)
- Track the count of manual instances in technical debt documentation
- When the threshold is reached, prioritize automation before adding the next instance
- Manual maintenance of N instances is O(N) ongoing cost; automation is a one-time investment