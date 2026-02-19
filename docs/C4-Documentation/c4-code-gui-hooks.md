# C4 Code Level: GUI Hooks

## Overview
- **Name**: GUI Custom React Hooks
- **Description**: Custom hooks providing data fetching, WebSocket connectivity, and utility functions for the GUI
- **Location**: `gui/src/hooks/`
- **Language**: TypeScript
- **Purpose**: Encapsulates side effects (API polling, WebSocket management, debouncing) and data fetching logic, keeping components focused on presentation
- **Parent Component**: [Web GUI](./c4-component-web-gui.md)

## Code Elements

### Functions/Methods

#### `useHealth(pollInterval: number): HealthState`
- **Location**: `gui/src/hooks/useHealth.ts`
- **Description**: Polls `/health/ready` at the given interval. Maps API responses to three statuses: `healthy` (ok response), `degraded` (503 with partial check failures), `unhealthy` (fetch error or non-ok)
- **Exports**: `HealthState` interface (`{ status: HealthStatus, checks: Record<string, any> }`), `HealthStatus` type (`'healthy' | 'degraded' | 'unhealthy'`)
- **Dependencies**: `react.useState`, `react.useEffect`

#### `useWebSocket(url: string): WebSocketHook`
- **Location**: `gui/src/hooks/useWebSocket.ts`
- **Description**: Manages a WebSocket connection with automatic reconnection using exponential backoff (base 1s, max 30s). Resets retry count on successful connection. Provides send function and last received message.
- **Exports**: `ConnectionState` type (`'connected' | 'disconnected' | 'reconnecting'`), `WebSocketHook` interface (`{ state, lastMessage, send }`)
- **Dependencies**: `react.useState`, `react.useEffect`, `react.useCallback`, `react.useRef`

#### `useMetrics(pollInterval: number): Metrics`
- **Location**: `gui/src/hooks/useMetrics.ts`
- **Description**: Polls `/metrics` (Prometheus text format) at the given interval. Parses request count and average duration.
- **Exported Function**: `parsePrometheus(text: string): Metrics` -- sums `http_requests_total` values, computes average from `http_request_duration_seconds_sum` / `_count`
- **Exports**: `Metrics` interface (`{ requestCount: number, avgDurationMs: number | null }`)
- **Dependencies**: `react.useState`, `react.useEffect`

#### `useDebounce<T>(value: T, delay?: number): T`
- **Location**: `gui/src/hooks/useDebounce.ts`
- **Description**: Generic debounce hook. Returns the debounced value after the specified delay (default 300ms). Resets timer on rapid changes.
- **Dependencies**: `react.useState`, `react.useEffect`

#### `useVideos(): UseVideosResult`
- **Location**: `gui/src/hooks/useVideos.ts`
- **Description**: Fetches video list from `/api/v1/videos` with search, sort, and pagination parameters. Watches `libraryStore` for changes and refetches.
- **Exports**: `Video` interface (id, path, filename, duration_frames, frame_rate_numerator/denominator, width, height, video_codec, audio_codec, file_size, thumbnail_path, created_at, updated_at), `UseVideosResult` interface
- **Dependencies**: `react.useState`, `react.useEffect`, `libraryStore`

#### `useProjects(): { projects, loading, error, refetch }`
- **Location**: `gui/src/hooks/useProjects.ts`
- **Description**: Fetches project list from `/api/v1/projects`. Also provides utility functions for project CRUD.
- **Exported Functions**:
  - `createProject(data): Promise<Project>` -- POST `/api/v1/projects`
  - `deleteProject(id): Promise<void>` -- DELETE `/api/v1/projects/{id}`
  - `fetchClips(projectId): Promise<Clip[]>` -- GET `/api/v1/projects/{id}/clips`
- **Exports**: `Project` interface (id, name, output_width, output_height, output_fps, created_at, updated_at), `Clip` interface (id, project_id, source_video_id, in_point, out_point, timeline_position, created_at, updated_at)
- **Dependencies**: `react.useState`, `react.useEffect`, `react.useCallback`

#### `useEffects(): { effects, loading, error, refetch }`
- **Location**: `gui/src/hooks/useEffects.ts`
- **Description**: Fetches effects list from `/api/v1/effects`. Returns array of Effect definitions.
- **Exported Function**: `deriveCategory(effectType: string): string` -- classifies effect types into categories: `audio` (volume, audio_*, acrossfade), `transition` (xfade), `video` (everything else)
- **Exports**: `Effect` interface (effect_type, name, description, parameter_schema, ai_hints, filter_preview)
- **Dependencies**: `react.useState`, `react.useEffect`, `react.useCallback`

#### `useEffectPreview(): void`
- **Location**: `gui/src/hooks/useEffectPreview.ts`
- **Description**: Connects the effect catalog store, form store, and preview store. When an effect is selected and parameters change, debounces (300ms) and calls POST `/api/v1/effects/preview` with `{ effect_type, parameters }`. Updates preview store with filter_string or error. Resets preview when no effect selected.
- **Dependencies**: `useDebounce`, `useEffectCatalogStore`, `useEffectFormStore`, `useEffectPreviewStore`

## Dependencies

### Internal Dependencies
- `gui/src/stores/libraryStore` -- used by `useVideos`
- `gui/src/stores/effectCatalogStore` -- used by `useEffectPreview`
- `gui/src/stores/effectFormStore` -- used by `useEffectPreview`
- `gui/src/stores/effectPreviewStore` -- used by `useEffectPreview`

### External Dependencies
- `react` (useState, useEffect, useCallback, useRef)

### API Endpoints

| Hook | Endpoint | Method | Polling |
|------|----------|--------|---------|
| useHealth | `/health/ready` | GET | Configurable interval |
| useMetrics | `/metrics` | GET | Configurable interval |
| useVideos | `/api/v1/videos` | GET | On store change |
| useProjects | `/api/v1/projects` | GET/POST/DELETE | On mount |
| useEffects | `/api/v1/effects` | GET | On mount |
| useEffectPreview | `/api/v1/effects/preview` | POST | Debounced on param change |

## Relationships

```mermaid
classDiagram
    class useHealth {
        +useHealth(pollInterval) HealthState
        polls /health/ready
    }
    class useWebSocket {
        +useWebSocket(url) WebSocketHook
        auto-reconnect
        exponential backoff
    }
    class useMetrics {
        +useMetrics(pollInterval) Metrics
        +parsePrometheus(text) Metrics
        polls /metrics
    }
    class useDebounce {
        +useDebounce~T~(value, delay) T
        generic debounce
    }
    class useVideos {
        +useVideos() UseVideosResult
        Video interface
    }
    class useProjects {
        +useProjects() result
        +createProject(data) Project
        +deleteProject(id) void
        +fetchClips(id) Clip[]
    }
    class useEffects {
        +useEffects() result
        +deriveCategory(type) string
        Effect interface
    }
    class useEffectPreview {
        +useEffectPreview() void
        POST /effects/preview
        debounced
    }

    class libraryStore {
        <<zustand>>
        searchQuery
        sortField
        sortOrder
    }
    class effectCatalogStore {
        <<zustand>>
        selectedEffect
    }
    class effectFormStore {
        <<zustand>>
        parameters
    }
    class effectPreviewStore {
        <<zustand>>
        filterString
    }

    useVideos --> libraryStore : watches
    useEffectPreview --> effectCatalogStore : reads selectedEffect
    useEffectPreview --> effectFormStore : reads parameters
    useEffectPreview --> effectPreviewStore : writes filterString
    useEffectPreview --> useDebounce : debounces params
```
