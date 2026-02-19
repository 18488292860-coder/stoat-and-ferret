# v007 Learnings Detail

## LRN-032: Schema-Driven Architecture Enables Backend-to-Frontend Consistency

**Tags:** pattern, architecture, schema, frontend, backend
**Source:** v007 cross-theme (T02-effect-registry-api/T03-effect-workshop-gui)

### Content

#### Context

When building a full-stack feature where the backend defines structured data (effect parameters, form fields, configuration options) and the frontend must render appropriate UI for that data.

#### Insight

Define data schemas once in the backend (e.g., JSON Schema) and use them to drive frontend UI generation. This eliminates hardcoded frontend components per data type and ensures both layers validate the same constraints. The schema becomes a contract between backend and frontend — adding a new data type requires only a new schema definition, not frontend code changes.

#### Evidence

In v007, JSON schemas defined in the effect registry (Theme 02) drove the frontend parameter form generator (Theme 03). The `SchemaField` dispatcher component routed each schema property to a typed sub-component based on `type` and `format` fields. All 9 effect types with different parameter schemas required zero frontend changes — the form generator rendered appropriate widgets (number with range slider, string, enum dropdown, boolean checkbox, color picker) automatically. Validation occurred at both layers using the same schema definitions.

#### Application

- Define data schemas in the backend using a standard format (JSON Schema, OpenAPI)
- Build frontend form/UI generators that consume schemas dynamically via a dispatcher pattern
- Test schema-driven rendering with isolated unit tests per widget type
- When adding new data types, add only the schema definition — no frontend changes needed

---

## LRN-033: Fix CI Reliability Before Dependent Development Cycles

**Tags:** process, ci, reliability, planning, failure-mode
**Source:** v007/03-effect-workshop-gui retrospective, v007/04-quality-validation retrospective, v007 version retrospective

### Content

#### Context

When planning a version or theme that builds on existing CI infrastructure (especially E2E tests), and there are known flaky or failing tests in the CI pipeline.

#### Insight

Fix CI reliability issues before starting development cycles that depend on CI for PR merges. A single flaky test can block PR merges across multiple features and themes, creating compounding friction. The cost of fixing CI upfront is far less than the cumulative cost of repeated investigation, retry attempts, and merge delays across multiple features.

#### Evidence

In v007, a pre-existing flaky E2E test (BL-055, `project-creation.spec.ts`) blocked PR merges for Theme 03 features 002 and 003, and was flagged again in Theme 04. Three CI retry attempts per feature, plus timeout increase investigation, yielded no resolution. The same test failed consistently on the `main` branch, confirming it was pre-existing. Two themes of development were affected by a single unresolved CI failure.

#### Application

- Before starting GUI-heavy or E2E-dependent development, audit CI pipeline health
- Fix or skip-mark known flaky tests before beginning new feature work
- Establish a CI health gate as a precondition for starting development versions
- Track flaky tests as high-priority backlog items, not low-priority tech debt

---

## LRN-034: Validate Numeric Requirements Against Upstream Sources During Design

**Tags:** process, requirements, design, validation, decision-framework
**Source:** v007/01-rust-filter-builders retrospective, v007/01-rust-filter-builders/002-transition-filter-builders completion-report

### Content

#### Context

When writing requirements or specifications that include numeric claims about external systems (e.g., "64 FFmpeg filter types", "12 supported codecs", "N API endpoints").

#### Insight

Cross-check numeric claims in requirements against upstream source documentation or code during the design phase, not during implementation. Discrepancies between requirements and reality create confusion during implementation and force deviation documentation. A 5-minute verification during design saves implementation-time surprises and spec deviation debates.

#### Evidence

In v007 Theme 01 Feature 002 (transition filter builders), the requirements specified "64 FFmpeg xfade variants" based on early research. During implementation, the actual FFmpeg source code (`libavfilter/vf_xfade.c`) contained 59 transition types (58 built-in + custom). The implementer had to decide whether to pad the enum to match the spec or implement the actual count, then document the deviation in the completion report.

#### Application

- When requirements reference counts or capabilities of external tools/libraries, verify against the authoritative source (source code, official docs)
- Include the verification source in the requirements document (e.g., "59 variants per FFmpeg libavfilter/vf_xfade.c")
- Flag numeric claims as "approximate" or "verified" in the design document
- Prefer linking to upstream source rather than hardcoding counts that may change

---

## LRN-035: Automate Manual Maintenance When Third Instance Appears

**Tags:** process, tooling, maintenance, automation, decision-framework
**Source:** v007/01-rust-filter-builders retrospective, v007 version retrospective

### Content

#### Context

When a manual maintenance task (stub generation, config syncing, schema updates) that was acceptable for 1-2 instances is about to grow to 3+ instances.

#### Insight

Apply the "rule of three" to manual maintenance tasks: manual is fine for 1-2 instances, but automate when a third appears. Each new manual instance increases the risk of inconsistency and the maintenance burden. Set an explicit threshold during the first manual instance so the automation trigger is pre-decided, not discovered through accumulated pain.

#### Evidence

In v007 Theme 01, `pyo3_stub_gen` does not support enum types, requiring manual stub entries in two separate `.pyi` files for each enum. FadeCurve (v006) and TransitionType (v007) both needed hand-maintained entries in both `src/stoat_ferret_core/_core.pyi` and `stubs/stoat_ferret_core/_core.pyi`. The Theme 01 retrospective explicitly recommended: "If a third enum type is needed, invest in automating enum stub generation." The v007 version retrospective listed this as medium-priority technical debt.

#### Application

- When introducing the first manual maintenance instance, document the automation trigger threshold (usually 3)
- Track the count of manual instances in technical debt documentation
- When the threshold is reached, prioritize automation before adding the next instance
- Manual maintenance of N instances is O(N) ongoing cost; automation is a one-time investment

---

## LRN-036: Debounced Stateless Preview Endpoints for Real-Time UI

**Tags:** pattern, api-design, frontend, performance, ux
**Source:** v007/03-effect-workshop-gui/003-live-filter-preview completion-report, v007/03-effect-workshop-gui retrospective

### Content

#### Context

When building a UI that needs to show a live preview (filter strings, computed results, rendered output) that updates as users change parameters.

#### Insight

Combine a stateless backend preview endpoint with client-side debouncing for responsive real-time previews without API flooding. The preview endpoint should require no persistent context (no project/session state) — just accept input parameters and return the computed result. Client-side debouncing (200-300ms) ensures only the final parameter state triggers an API call during rapid changes.

#### Evidence

In v007 Theme 03 Feature 003, a `POST /effects/preview` endpoint accepts `{effect_type, parameters}` and returns `{filter_string}`. It validates via the registry and calls `build_fn` with no side effects or persistence. The frontend `useEffectPreview` hook debounces at 300ms using an existing `useDebounce` hook. Users see filter strings update as they adjust sliders without the backend receiving per-keystroke API calls.

#### Application

- Create a stateless preview endpoint that takes input and returns computed output with no side effects
- Apply 200-300ms client-side debounce to prevent rapid-fire API calls
- Reuse existing debounce utilities rather than implementing custom timers
- Keep the preview endpoint lightweight: validation + computation only, no persistence
- Show loading indicators during the debounce+fetch cycle for user feedback

---

## LRN-037: Feature Composition Succeeds Through Independent Store Interfaces

**Tags:** pattern, frontend, architecture, composability, zustand
**Source:** v007/03-effect-workshop-gui retrospective, v007 version retrospective

### Content

#### Context

When building a multi-feature UI workflow where each feature is developed sequentially but must compose into a unified experience.

#### Insight

Design each feature as a standalone component with its own independent state store and clean public interface (selectors and actions). When the composition feature arrives, it can orchestrate existing components without modifying their internals. Each store must be self-contained, and inter-component communication should flow through the orchestrator, not through direct store cross-references.

#### Evidence

In v007 Theme 03, four features were developed sequentially: catalog (F001), parameter form (F002), filter preview (F003), and builder workflow (F004). Each created its own Zustand store (effectCatalogStore, effectFormStore, effectPreviewStore, effectStackStore). When Feature 004 composed all components into the EffectsPage workflow, no prior features required modification. The preview hook watched form store state reactively; the workflow orchestrator connected catalog selection to form initialization to preview display. The retrospective noted: "Feature composition in 004 was smooth because each prior feature was designed as a standalone component with clean store interfaces."

#### Application

- Give each feature its own state store with clearly defined selectors and actions
- Avoid direct cross-store imports between features; let the orchestrator mediate
- Design stores to be reactive (selectors that other components can subscribe to)
- Test each feature component in isolation before composing
- Plan the composition feature last in the sequence so all dependencies are stable

---

## LRN-038: Sub-Agent File Operations Fail When Read-Before-Write Is Skipped

**Tags:** process, tooling, sub-agents, failure-mode, session-health
**Source:** session analytics (v007 period)

### Content

#### Context

When using AI sub-agents (Task tool spawns) to perform file operations as part of automated workflows, including documentation generation and code analysis.

#### Insight

Sub-agents frequently attempt to Write or Edit files they haven't Read first, causing repeated "File has not been read yet" errors. This wastes API round-trips and can cascade into sibling tool call failures when parallel operations are used. Sub-agent prompts should explicitly instruct reading target files before any write/edit operations, and the Read step should be a prerequisite in any file modification workflow.

#### Evidence

Session analytics showed recurring patterns across multiple sub-agent sessions: `agent-aaf973a` had 3 Write errors, `agent-a2d8a40` had 8+ Edit errors, `agent-ad2d48f` had 3 Write errors, `agent-af73f3f` had 2 Write errors — all "File has not been read yet" failures. These appeared across C4 documentation generation, exploration tasks, and analysis sub-agents. The pattern represents wasted API round-trips where sub-agents attempted to create/modify output files without the required prior Read call.

#### Application

- Include explicit "Read before Write/Edit" instructions in sub-agent prompts
- Structure sub-agent workflows as: Read target file -> process -> Write/Edit result
- When creating new output files, note that Write still requires a prior Read attempt
- Monitor sub-agent error rates for this pattern as a quality signal for prompt effectiveness

---

## LRN-039: Excessive Context Compaction Signals Need for Task Decomposition

**Tags:** process, session-health, context-management, decomposition, efficiency
**Source:** session analytics (v007 period), v007 session health findings

### Content

#### Context

When AI agent sessions are long-running and involve reading many files, processing large codebases, or performing multi-step implementation tasks.

#### Insight

High context compaction frequency (3+ events per session, with some sessions hitting 10-16) indicates sessions are consuming context faster than productive work requires. This suggests tasks should be decomposed into smaller sub-tasks, file reading should be more targeted, or intermediate results should be summarized before moving to the next step. When nearly half of sessions exhibit this pattern, it is systemic rather than incidental.

#### Evidence

Session health analysis found 24 of ~50 sessions (48%) had 3+ context compaction events. The worst session had 16 compaction events, meaning it exhausted and compressed its context 16 times. Sessions involving C4 documentation generation, multi-feature implementation, and large codebase exploration were most affected. The finding suggests sessions reading many large files without summarizing routinely exceed context limits.

#### Application

- Decompose large tasks into focused sub-tasks that each fit within context limits
- Summarize intermediate results before moving to the next phase of work
- Use targeted file reading (specific sections, line ranges) instead of full-file reads when possible
- Monitor compaction count as a health metric: sessions with 5+ compactions likely benefit from decomposition
- Delegate read-heavy research phases to sub-agents to protect the main session context
