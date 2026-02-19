# v007 Learning Extraction

Extracted 8 new learnings (LRN-032 through LRN-039) from 11 completion reports, 5 retrospectives, session health findings, and session analytics. Identified 9 existing learnings reinforced by v007 evidence. Filtered out 8 candidate items that were too implementation-specific, tracked as backlog items, or duplicated existing learnings.

## New Learnings

| ID | Title | Tags | Source |
|----|-------|------|--------|
| LRN-032 | Schema-Driven Architecture Enables Backend-to-Frontend Consistency | pattern, architecture, schema, frontend, backend | v007 cross-theme (T02/T03) |
| LRN-033 | Fix CI Reliability Before Dependent Development Cycles | process, ci, reliability, planning, failure-mode | v007/T03 retrospective, v007/T04 retrospective, v007 version retrospective |
| LRN-034 | Validate Numeric Requirements Against Upstream Sources During Design | process, requirements, design, validation, decision-framework | v007/T01 retrospective, v007/T01-F002 completion-report |
| LRN-035 | Automate Manual Maintenance When Third Instance Appears | process, tooling, maintenance, automation, decision-framework | v007/T01 retrospective, v007 version retrospective |
| LRN-036 | Debounced Stateless Preview Endpoints for Real-Time UI | pattern, api-design, frontend, performance, ux | v007/T03-F003 completion-report, v007/T03 retrospective |
| LRN-037 | Feature Composition Succeeds Through Independent Store Interfaces | pattern, frontend, architecture, composability, zustand | v007/T03 retrospective, v007 version retrospective |
| LRN-038 | Sub-Agent File Operations Fail When Read-Before-Write Is Skipped | process, tooling, sub-agents, failure-mode, session-health | session analytics (v007 period) |
| LRN-039 | Excessive Context Compaction Signals Need for Task Decomposition | process, session-health, context-management, decomposition, efficiency | session analytics, v007 session health findings |

## Reinforced Learnings

| ID | Title | New Evidence |
|----|-------|-------------|
| LRN-028 | Repeatable Implementation Templates Across Feature Series | v007 T01: 7 new Rust builders used same template pattern from v006; all 42/42 acceptance criteria passed |
| LRN-025 | Feature Handoff Documents Enable Zero-Rework Sequencing | v007 T02: Handoff between F001 (registry refactor) and F002 (transition endpoint) enabled zero-rework build |
| LRN-024 | Focused Zustand Stores Over Monolithic State Management | v007 T03: 4 new focused Zustand stores (catalog, form, preview, stack) followed the established pattern |
| LRN-023 | Client-Side Navigation for SPA E2E Tests Without Server-Side Routing | v007 T04: E2E effect workshop tests used client-side navigation workaround; documented in API spec as known limitation |
| LRN-022 | Automated Accessibility Testing Catches Real Violations | v007 T04: axe-core WCAG AA scans on effect catalog and parameter form found zero violations |
| LRN-030 | Architecture Documentation as an Explicit Feature Deliverable | v007 T02-F003 and T04-F002: Both dedicated documentation features efficient when done immediately after implementation |
| LRN-007 | Parity Tests Prevent Test Double Drift | v007 T02: Parity tests comparing new registry dispatch output against hardcoded expected strings caught regressions during refactor |
| LRN-031 | Detailed Design Specifications Correlate with First-Iteration Success | v007: 131/131 acceptance criteria passed across all 11 features; all first-iteration completions |
| LRN-029 | Conscious Simplicity with Documented Upgrade Paths | v007 T02/T03: Array-index-based CRUD and JSON column storage chose simplest approach with documented upgrade triggers |

## Filtered Out

**Total filtered: 8 candidates**

| Category | Count | Examples |
|----------|-------|---------|
| Too implementation-specific | 3 | FFmpeg variant count details (59 vs 64), DuckingPattern composition API validation, specific file line counts |
| Tracked as backlog/tech debt | 3 | BL-055 flaky E2E test, SPA fallback missing, definitions.py module size |
| Absorbed into existing learnings | 2 | Array-index CRUD (absorbed into LRN-029 reinforcement), Zustand store pattern (absorbed into LRN-024 reinforcement) |
