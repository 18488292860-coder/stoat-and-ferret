# External Research — v005

## 1. Frontend Framework Selection: React vs Svelte

**Recommendation: React 18+ with TypeScript**

| Criterion | React | Svelte | Winner |
|-----------|-------|--------|--------|
| TypeScript support | Excellent, mature | Good, improving | React |
| Component libraries | Vast ecosystem | Smaller ecosystem | React |
| Virtual scrolling libs | react-window, react-virtuoso, tanstack-virtual | Limited options | React |
| State management | Zustand (recommended in design docs) | Built-in stores | Tie |
| Developer familiarity | Most popular framework | Growing but niche | React |
| Bundle size | Larger runtime | Smaller runtime | Svelte |
| Design doc alignment | JSX examples throughout 08-gui-architecture.md | Not referenced | React |

React wins 5/7 criteria. The design documents already use React patterns (JSX, hooks, Zustand). Svelte's smaller bundle is not significant for a locally-served app.

## 2. Vite Project Setup (Source: DeepWiki vitejs/vite)

**Scaffolding:**
```bash
npm create vite@latest gui -- --template react-ts
```

**Dev Server Proxy Configuration:**
```typescript
// gui/vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:8000',
      '/health': 'http://localhost:8000',
      '/metrics': 'http://localhost:8000',
      '/ws': {
        target: 'ws://localhost:8000',
        ws: true,
      },
    },
  },
  base: '/gui/',
  build: {
    outDir: 'dist',
  },
})
```

**Production Build:**
- `base: '/gui/'` sets asset URL prefix for sub-path deployment
- `build.outDir` defaults to `dist` (keep default)
- `build.manifest: true` optional — generates `.vite/manifest.json` for backend asset resolution
- Code splitting via `build.rollupOptions.output` for route-based chunks

**Source:** [DeepWiki vitejs/vite Configuration System](https://deepwiki.com/wiki/vitejs/vite#3.1)

## 3. FastAPI Static File Serving (Source: Web Search)

**Basic SPA Mount:**
```python
from fastapi.staticfiles import StaticFiles

app.mount("/gui", StaticFiles(directory="gui/dist", html=True), name="gui")
```

The `html=True` flag enables:
- Serves `index.html` for directory requests
- Falls back to `index.html` for unknown paths (SPA client-side routing)

**Route Ordering:** Mount StaticFiles **after** all API routers to prevent path conflicts. FastAPI evaluates routes in order; API routes must take priority over the catch-all static mount.

**Custom SPA Handler (if html=True insufficient):**
```python
class SPAStaticFiles(StaticFiles):
    async def get_response(self, path, scope):
        response = await super().get_response(path, scope)
        if response.status_code == 404:
            response = await super().get_response("index.html", scope)
        return response
```

**Sources:**
- [FastAPI Static Files docs](https://fastapi.tiangolo.com/tutorial/static-files/)
- [Serving React with FastAPI](https://davidmuraya.com/blog/serving-a-react-frontend-application-with-fastapi/)

## 4. FastAPI WebSocket Patterns (Source: Web Search)

**Basic Endpoint:**
```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_json({"type": "echo", "data": data})
    except WebSocketDisconnect:
        pass
```

**ConnectionManager Pattern:**
```python
class ConnectionManager:
    def __init__(self):
        self.active_connections: set[WebSocket] = set()

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.add(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.discard(websocket)

    async def broadcast(self, message: dict):
        dead = []
        for conn in self.active_connections:
            try:
                await conn.send_json(message)
            except Exception:
                dead.append(conn)
        for conn in dead:
            self.active_connections.discard(conn)
```

**Best Practices:**
- Use `set()` not `list()` for O(1) add/remove
- Wrap broadcast sends in try/except to handle dead connections
- Use `asyncio.Lock` if concurrent broadcasts are possible
- Inject via `create_app(ws_manager=...)` per LRN-005
- Heartbeat: periodic ping/pong or application-level keepalive

**Sources:**
- [FastAPI WebSockets Guide 2025](https://dev-faizan.medium.com/building-real-time-applications-with-fastapi-websockets-a-complete-guide-2025-40f29d327733)
- [Managing Multiple WebSocket Clients](https://hexshift.medium.com/managing-multiple-websocket-clients-in-fastapi-ce5b134568a2)
- [FastAPI WebSockets docs](https://fastapi.tiangolo.com/advanced/websockets/)

## 5. FFmpeg Thumbnail Extraction (Source: Web Search)

**Single Frame Extraction:**
```bash
ffmpeg -ss 5 -i input.mp4 -frames:v 1 -vf "scale=320:-1" -q:v 5 thumbnail.jpg
```

- `-ss 5` before `-i`: fast seeking to 5-second mark
- `-frames:v 1`: extract exactly 1 frame
- `-vf "scale=320:-1"`: scale to 320px width, maintain aspect ratio
- `-q:v 5`: JPEG quality (2=best, 31=worst; 5 is good balance)

**Integration with Existing Executor:**
```python
result = executor.run([
    "-ss", "5",
    "-i", video_path,
    "-frames:v", "1",
    "-vf", "scale=320:-1",
    "-q:v", "5",
    "-y",  # overwrite
    output_path,
])
```

Fits perfectly into `RealFFmpegExecutor` / `RecordingFFmpegExecutor` / `FakeFFmpegExecutor` pattern.

**Sources:**
- [Extract thumbnails with FFmpeg (Mux)](https://www.mux.com/articles/extract-thumbnails-from-a-video-with-ffmpeg)
- [FFmpeg Thumbnail Techniques](https://ottverse.com/thumbnails-screenshots-using-ffmpeg/)

## 6. Playwright E2E Testing (Source: DeepWiki microsoft/playwright)

**Setup:**
```bash
cd gui
npm init playwright@latest
```

**Configuration (`playwright.config.ts`):**
```typescript
export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:8000/gui/',
    trace: 'on-first-retry',
  },
  webServer: {
    command: 'cd .. && uvicorn src.stoat_ferret.api.app:create_app --factory',
    url: 'http://localhost:8000/health/live',
    reuseExistingServer: !process.env.CI,
  },
})
```

**WebSocket Testing:**
```typescript
await page.routeWebSocket('/ws', ws => {
  ws.onMessage(message => {
    if (message === 'request') ws.send('response');
  });
});
```

**Accessibility (WCAG AA):**
```typescript
import AxeBuilder from '@axe-core/playwright';

const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();
expect(results.violations).toEqual([]);
```

**CI GitHub Actions Workflow:**
1. Checkout, setup Node.js
2. `npm ci` + `npx playwright install --with-deps`
3. Build frontend: `npm run build`
4. Run tests: `npx playwright test`
5. Upload HTML report as artifact

**Sources:**
- [DeepWiki microsoft/playwright](https://deepwiki.com/wiki/microsoft/playwright)
- [@axe-core/playwright docs](https://www.npmjs.com/package/@axe-core/playwright)
