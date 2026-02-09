# THEME_INDEX - v005

## 01-frontend-foundation

Scaffold the frontend project, configure static file serving from FastAPI, set up CI pipeline integration, and add WebSocket real-time event support.

**Features:**

- 001-frontend-scaffolding: Scaffold React/Vite/TypeScript project in gui/, configure FastAPI static file serving, and update CI pipeline
- 002-websocket-endpoint: Implement /ws WebSocket endpoint with ConnectionManager, event broadcasting, and correlation ID support
- 003-settings-and-docs: Add new settings fields and update architecture/API/AGENTS documentation

## 02-backend-services

Add backend capabilities that the GUI components consume: thumbnail generation pipeline and pagination total count fix.

**Features:**

- 001-thumbnail-pipeline: Implement thumbnail generation service using FFmpeg executor pattern with API endpoint and scan integration
- 002-pagination-total-count: Add count() to repository protocol and fix list/search endpoints to return true total

## 03-gui-components

Build the four main GUI panels: application shell with navigation, dashboard, library browser, and project manager.

**Features:**

- 001-application-shell: Build application shell with navigation tabs, health indicator, status bar, and progressive tab disclosure
- 002-dashboard-panel: Build dashboard with health cards, real-time activity log via WebSocket, and metrics overview
- 003-library-browser: Build library browser with video grid, thumbnails, search, sort/filter, scan modal, and virtual scrolling
- 004-project-manager: Build project manager with list, creation modal, details view with timeline positions, and delete confirmation

## 04-e2e-testing

Establish Playwright E2E test infrastructure with CI integration, covering navigation, scan trigger, project creation, and WCAG AA accessibility checks.

**Features:**

- 001-playwright-setup: Configure Playwright with CI integration, webServer config for FastAPI, and browser installation
- 002-e2e-test-suite: Write E2E tests for navigation, scan trigger, project creation, and WCAG AA accessibility checks
