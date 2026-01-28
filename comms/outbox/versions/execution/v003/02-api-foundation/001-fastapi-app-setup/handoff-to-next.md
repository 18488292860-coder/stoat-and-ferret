# Handoff: 001-fastapi-app-setup -> Next Feature

## What Was Built

A minimal FastAPI application factory with:
- `create_app()` function that returns configured FastAPI instance
- Lifespan context manager for database connection management
- Entry point for server startup via `python -m stoat_ferret.api`
- Settings module for configuration
- Test infrastructure in `tests/test_api/`

## Key Patterns Established

### Application Factory
```python
from stoat_ferret.api.app import create_app
app = create_app()
```

### Accessing Database in Routes
```python
from fastapi import Request

@router.get("/")
async def endpoint(request: Request):
    db = request.app.state.db
    # Use db connection...
```

### Testing API Endpoints
```python
# tests/test_api/conftest.py provides:
# - app: FastAPI instance with test settings
# - client: TestClient for HTTP requests

def test_endpoint(client: TestClient):
    response = client.get("/api/v1/endpoint")
    assert response.status_code == 200
```

## Ready for Next Features

The following directories are prepared and ready for content:
- `src/stoat_ferret/api/routers/` - Add route modules here
- `src/stoat_ferret/api/schemas/` - Add Pydantic request/response models here
- `src/stoat_ferret/api/services/` - Add business logic here
- `src/stoat_ferret/api/middleware/` - Add middleware here

## Suggested Next Steps

1. **Health Check Endpoint**: Add `/api/v1/health` endpoint to verify service is running
2. **Video Router**: Create `routers/videos.py` for video CRUD operations
3. **Error Handling Middleware**: Add standardized error responses
4. **Request Validation**: Add Pydantic schemas for request validation

## Dependencies Added

- `fastapi>=0.109` - Web framework
- `uvicorn[standard]>=0.27` - ASGI server with WebSocket and HTTP/2 support

These are already in `pyproject.toml` and available in the venv.
