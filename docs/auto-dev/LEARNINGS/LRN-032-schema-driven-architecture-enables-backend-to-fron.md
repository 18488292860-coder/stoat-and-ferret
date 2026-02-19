## Context

When building a full-stack feature where the backend defines structured data (effect parameters, form fields, configuration options) and the frontend must render appropriate UI for that data.

## Insight

Define data schemas once in the backend (e.g., JSON Schema) and use them to drive frontend UI generation. This eliminates hardcoded frontend components per data type and ensures both layers validate the same constraints. The schema becomes a contract between backend and frontend — adding a new data type requires only a new schema definition, not frontend code changes.

## Evidence

In v007, JSON schemas defined in the effect registry (Theme 02) drove the frontend parameter form generator (Theme 03). The `SchemaField` dispatcher component routed each schema property to a typed sub-component based on `type` and `format` fields. All 9 effect types with different parameter schemas required zero frontend changes — the form generator rendered appropriate widgets (number with range slider, string, enum dropdown, boolean checkbox, color picker) automatically. Validation occurred at both layers using the same schema definitions.

## Application

- Define data schemas in the backend using a standard format (JSON Schema, OpenAPI)
- Build frontend form/UI generators that consume schemas dynamically via a dispatcher pattern
- Test schema-driven rendering with isolated unit tests per widget type
- When adding new data types, add only the schema definition — no frontend changes needed