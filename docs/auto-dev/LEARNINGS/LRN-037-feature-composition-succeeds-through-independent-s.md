## Context

When building a multi-feature UI workflow where each feature is developed sequentially but must compose into a unified experience.

## Insight

Design each feature as a standalone component with its own independent state store and clean public interface (selectors and actions). When the composition feature arrives, it can orchestrate existing components without modifying their internals. Each store must be self-contained, and inter-component communication should flow through the orchestrator, not through direct store cross-references.

## Evidence

In v007 Theme 03, four features were developed sequentially: catalog (F001), parameter form (F002), filter preview (F003), and builder workflow (F004). Each created its own Zustand store (effectCatalogStore, effectFormStore, effectPreviewStore, effectStackStore). When Feature 004 composed all components into the EffectsPage workflow, no prior features required modification. The preview hook watched form store state reactively; the workflow orchestrator connected catalog selection to form initialization to preview display. The retrospective noted: "Feature composition in 004 was smooth because each prior feature was designed as a standalone component with clean store interfaces."

## Application

- Give each feature its own state store with clearly defined selectors and actions
- Avoid direct cross-store imports between features; let the orchestrator mediate
- Design stores to be reactive (selectors that other components can subscribe to)
- Test each feature component in isolation before composing
- Plan the composition feature last in the sequence so all dependencies are stable