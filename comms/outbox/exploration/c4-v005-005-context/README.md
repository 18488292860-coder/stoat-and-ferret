# C4 Context Documentation — stoat-and-ferret

Identified 2 personas, documented 10 system features, and created 3 user journeys for a self-hosted, non-destructive video editor with hybrid Python/Rust architecture.

- **System Purpose:** Browser-based video editor that organizes video libraries, defines frame-accurate clips, and builds type-safe FFmpeg commands.
- **Personas Identified:** 2 — Editor User (human), Prometheus Monitoring (external system, optional)
- **Features Documented:** 10 — Video Library, Directory Scanning, Thumbnail Generation, Project Management, Clip Editing, FFmpeg Command Building, Real-time Dashboard, Full-text Search, Prometheus Metrics, Audit Logging
- **User Journeys Created:** 3 — Video Library browsing, Project and Clip editing, Prometheus integration
- **External Dependencies:** 2 — FFmpeg/ffprobe (required, subprocess), Prometheus (optional, HTTP scrape)
- **Sources Used:** c4-container.md, c4-component.md, README.md, AGENTS.md, PLAN.md, ~55 Python test files (feature discovery), backlog items (BL-001 through BL-052)
