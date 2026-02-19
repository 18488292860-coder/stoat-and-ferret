# C4 Code Level: GUI Stores

## Overview
- **Name**: GUI Zustand Stores
- **Description**: Global state management stores using Zustand for the GUI application
- **Location**: `gui/src/stores/`
- **Language**: TypeScript
- **Purpose**: Provides lightweight, hook-based global state management for UI concerns (search/sort/filter state, form state, effect data) without prop drilling

## Code Elements

### Stores

#### `activityStore.ts`
- **Store**: `useActivityStore`
- **State**:
  - `entries: ActivityEntry[]` -- WebSocket event log entries
- **Actions**:
  - `addEntry(entry: ActivityEntry): void` -- prepends entry, enforces max 50 with FIFO eviction
- **Types**:
  - `ActivityEntry { type: string, timestamp: string, details: Record<string, unknown> }`
- **Consumers**: `ActivityLog` component

#### `libraryStore.ts`
- **Store**: `useLibraryStore`
- **State**:
  - `searchQuery: string` -- current video search text
  - `sortField: 'date' | 'name' | 'duration'` -- sort column
  - `sortOrder: 'asc' | 'desc'` -- sort direction
  - `page: number` -- current page (1-based)
  - `pageSize: number` -- items per page (default 20)
- **Actions**:
  - `setSearchQuery(q: string): void`
  - `setSortField(field): void`
  - `setSortOrder(order): void`
  - `setPage(page: number): void`
- **Consumers**: `LibraryPage`, `useVideos` hook

#### `projectStore.ts`
- **Store**: `useProjectStore`
- **State**:
  - `selectedProjectId: string | null` -- currently viewed project
  - `createModalOpen: boolean` -- create project modal visibility
  - `deleteModalOpen: boolean` -- delete confirmation modal visibility
- **Actions**:
  - `setSelectedProjectId(id: string | null): void`
  - `setCreateModalOpen(open: boolean): void`
  - `setDeleteModalOpen(open: boolean): void`
- **Consumers**: `ProjectsPage`

#### `effectCatalogStore.ts`
- **Store**: `useEffectCatalogStore`
- **State**:
  - `searchQuery: string` -- effect search text
  - `selectedCategory: string | null` -- active category filter
  - `selectedEffect: string | null` -- currently selected effect type
  - `viewMode: 'grid' | 'list'` -- catalog display mode
- **Actions**:
  - `setSearchQuery(q: string): void`
  - `setCategory(cat: string | null): void`
  - `selectEffect(effect: string | null): void`
  - `toggleViewMode(): void`
- **Consumers**: `EffectCatalog` component, `EffectsPage`, `useEffectPreview` hook

#### `effectFormStore.ts`
- **Store**: `useEffectFormStore`
- **State**:
  - `schema: ParameterSchema | null` -- JSON Schema for current effect
  - `parameters: Record<string, unknown>` -- current form values
  - `validationErrors: Record<string, string>` -- field-level errors
  - `isDirty: boolean` -- whether form has been modified
- **Actions**:
  - `setSchema(schema: ParameterSchema): void` -- sets schema, initializes defaults via `defaultsFromSchema()`, clears errors
  - `setParameter(key: string, value: unknown): void` -- sets single param, marks dirty
  - `setValidationErrors(errors: Record<string, string>): void`
  - `resetForm(): void` -- clears schema, params, errors, isDirty
- **Types**:
  - `SchemaProperty { type: string, description?: string, default?: unknown, minimum?: number, maximum?: number, enum?: string[], format?: string }`
  - `ParameterSchema { type: 'object', properties: Record<string, SchemaProperty> }`
- **Helper**: `defaultsFromSchema(schema: ParameterSchema): Record<string, unknown>` -- extracts default values from schema properties
- **Consumers**: `EffectParameterForm` component, `EffectsPage`, `useEffectPreview` hook

#### `effectPreviewStore.ts`
- **Store**: `useEffectPreviewStore`
- **State**:
  - `filterString: string` -- generated FFmpeg filter string
  - `isLoading: boolean` -- preview request in flight
  - `error: string | null` -- preview error message
- **Actions**:
  - `setFilterString(s: string): void`
  - `setLoading(loading: boolean): void`
  - `setError(error: string | null): void`
  - `reset(): void` -- clears all fields
- **Consumers**: `FilterPreview` component, `useEffectPreview` hook

#### `effectStackStore.ts`
- **Store**: `useEffectStackStore`
- **State**:
  - `selectedClipId: string | null` -- clip targeted for effects
  - `effects: AppliedEffect[]` -- effects applied to selected clip
  - `isLoading: boolean` -- fetch in flight
  - `error: string | null` -- last error
- **Actions**:
  - `selectClip(clipId: string | null): void` -- sets clip, clears effects/error
  - `setEffects(effects: AppliedEffect[]): void`
  - `setLoading(loading: boolean): void`
  - `setError(error: string | null): void`
  - `fetchEffects(projectId: string, clipId: string): Promise<void>` -- GET `/api/v1/projects/{id}/clips`, finds clip, extracts effects array
  - `removeEffect(projectId: string, clipId: string, index: number): Promise<void>` -- DELETE `/api/v1/projects/{id}/clips/{id}/effects/{idx}`, then refetches
  - `reset(): void` -- clears all state
- **Types**:
  - `AppliedEffect { effect_type: string, parameters: Record<string, unknown>, filter_string: string }`
- **Consumers**: `EffectsPage`, `EffectStack` component

## Dependencies

### Internal Dependencies
- None (stores are leaf modules with no internal imports)

### External Dependencies
- `zustand` (create function)

## Store Relationship Summary

| Store | State Fields | Actions | Primary Consumers |
|-------|-------------|---------|-------------------|
| activityStore | 1 | 1 | ActivityLog |
| libraryStore | 5 | 4 | LibraryPage, useVideos |
| projectStore | 3 | 3 | ProjectsPage |
| effectCatalogStore | 4 | 4 | EffectCatalog, EffectsPage, useEffectPreview |
| effectFormStore | 4 | 4 | EffectParameterForm, EffectsPage, useEffectPreview |
| effectPreviewStore | 3 | 4 | FilterPreview, useEffectPreview |
| effectStackStore | 4 | 7 | EffectsPage, EffectStack |

## Relationships

```mermaid
classDiagram
    class activityStore {
        +entries: ActivityEntry[]
        +addEntry(entry)
    }
    class libraryStore {
        +searchQuery: string
        +sortField: string
        +sortOrder: string
        +page: number
        +pageSize: number
        +setSearchQuery(q)
        +setSortField(f)
        +setSortOrder(o)
        +setPage(p)
    }
    class projectStore {
        +selectedProjectId: string?
        +createModalOpen: boolean
        +deleteModalOpen: boolean
    }
    class effectCatalogStore {
        +searchQuery: string
        +selectedCategory: string?
        +selectedEffect: string?
        +viewMode: grid|list
        +selectEffect(e)
        +toggleViewMode()
    }
    class effectFormStore {
        +schema: ParameterSchema?
        +parameters: Record
        +validationErrors: Record
        +isDirty: boolean
        +setSchema(s)
        +setParameter(k, v)
        +resetForm()
    }
    class effectPreviewStore {
        +filterString: string
        +isLoading: boolean
        +error: string?
        +setFilterString(s)
        +reset()
    }
    class effectStackStore {
        +selectedClipId: string?
        +effects: AppliedEffect[]
        +isLoading: boolean
        +error: string?
        +fetchEffects(proj, clip)
        +removeEffect(proj, clip, idx)
    }

    class EffectsPage {
        <<page>>
    }
    class useEffectPreview {
        <<hook>>
    }

    EffectsPage --> effectCatalogStore : reads selectedEffect
    EffectsPage --> effectFormStore : reads parameters
    EffectsPage --> effectStackStore : manages effects
    useEffectPreview --> effectCatalogStore : reads selectedEffect
    useEffectPreview --> effectFormStore : reads parameters
    useEffectPreview --> effectPreviewStore : writes filterString
```
