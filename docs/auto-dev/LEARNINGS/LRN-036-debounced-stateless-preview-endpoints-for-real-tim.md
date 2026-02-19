## Context

When building a UI that needs to show a live preview (filter strings, computed results, rendered output) that updates as users change parameters.

## Insight

Combine a stateless backend preview endpoint with client-side debouncing for responsive real-time previews without API flooding. The preview endpoint should require no persistent context (no project/session state) — just accept input parameters and return the computed result. Client-side debouncing (200-300ms) ensures only the final parameter state triggers an API call during rapid changes.

## Evidence

In v007 Theme 03 Feature 003, a `POST /effects/preview` endpoint accepts `{effect_type, parameters}` and returns `{filter_string}`. It validates via the registry and calls `build_fn` with no side effects or persistence. The frontend `useEffectPreview` hook debounces at 300ms using an existing `useDebounce` hook. Users see filter strings update as they adjust sliders without the backend receiving per-keystroke API calls.

## Application

- Create a stateless preview endpoint that takes input and returns computed output with no side effects
- Apply 200-300ms client-side debounce to prevent rapid-fire API calls
- Reuse existing debounce utilities rather than implementing custom timers
- Keep the preview endpoint lightweight: validation + computation only, no persistence
- Show loading indicators during the debounce+fetch cycle for user feedback