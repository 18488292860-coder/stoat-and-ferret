# Theme Index: v009

## 01-observability-pipeline

Wire the three observability components that exist as dead code into the application's DI chain and startup sequence.

**Features:**

- 001-ffmpeg-observability: Wire ObservableFFmpegExecutor into DI so FFmpeg operations emit metrics and structured logs
- 002-audit-logging: Wire AuditLogger into repository DI with a separate sync connection for audit entries
- 003-file-logging: Add RotatingFileHandler to configure_logging() for persistent log output to rotating files

## 02-gui-runtime-fixes

Fix three runtime gaps in the GUI and API layer: SPA routing, pagination totals, and WebSocket broadcasts.

**Features:**

- 001-spa-routing: Replace StaticFiles mount with catch-all route serving both static files and SPA fallback
- 002-pagination-fix: Add count() to AsyncProjectRepository, fix total in API, and add frontend pagination UI
- 003-websocket-broadcasts: Wire broadcast() into scan service and project router for real-time WebSocket events
