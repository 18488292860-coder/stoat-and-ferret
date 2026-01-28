# Implementation Plan: Clips API

## Step 1: Create Schemas
Create `src/stoat_ferret/api/schemas/project.py`:

```python
"""Project API schemas."""

from datetime import datetime
from pydantic import BaseModel, Field


class ProjectCreate(BaseModel):
    """Create project request."""
    name: str = Field(..., min_length=1)
    output_width: int = Field(default=1920, ge=1)
    output_height: int = Field(default=1080, ge=1)
    output_fps: int = Field(default=30, ge=1, le=120)


class ProjectResponse(BaseModel):
    """Project response."""
    id: str
    name: str
    output_width: int
    output_height: int
    output_fps: int
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True


class ProjectListResponse(BaseModel):
    """Project list response."""
    projects: list[ProjectResponse]
    total: int
```

Create `src/stoat_ferret/api/schemas/clip.py`:

```python
"""Clip API schemas."""

from datetime import datetime
from pydantic import BaseModel, Field


class ClipCreate(BaseModel):
    """Create clip request."""
    source_video_id: str
    in_point: int = Field(..., ge=0)
    out_point: int = Field(..., ge=0)
    timeline_position: int = Field(..., ge=0)


class ClipUpdate(BaseModel):
    """Update clip request."""
    in_point: int | None = Field(default=None, ge=0)
    out_point: int | None = Field(default=None, ge=0)
    timeline_position: int | None = Field(default=None, ge=0)


class ClipResponse(BaseModel):
    """Clip response."""
    id: str
    project_id: str
    source_video_id: str
    in_point: int
    out_point: int
    timeline_position: int
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True


class ClipListResponse(BaseModel):
    """Clip list response."""
    clips: list[ClipResponse]
    total: int
```

## Step 2: Create Projects Router
Create `src/stoat_ferret/api/routers/projects.py`:

```python
"""Project and clip endpoints."""

from datetime import datetime, timezone

from fastapi import APIRouter, Depends, HTTPException, Query, Request, Response, status
from stoat_ferret_core import FrameRate

from stoat_ferret.api.schemas.clip import ClipCreate, ClipListResponse, ClipResponse, ClipUpdate
from stoat_ferret.api.schemas.project import ProjectCreate, ProjectListResponse, ProjectResponse
from stoat_ferret.db.clip_repository import AsyncClipRepository, AsyncSQLiteClipRepository
from stoat_ferret.db.models import Clip, Project
from stoat_ferret.db.project_repository import AsyncProjectRepository, AsyncSQLiteProjectRepository

router = APIRouter(prefix="/api/v1/projects", tags=["projects"])


async def get_project_repository(request: Request) -> AsyncProjectRepository:
    return AsyncSQLiteProjectRepository(request.app.state.db)


async def get_clip_repository(request: Request) -> AsyncClipRepository:
    return AsyncSQLiteClipRepository(request.app.state.db)


@router.get("", response_model=ProjectListResponse)
async def list_projects(
    limit: int = Query(default=20, ge=1, le=100),
    offset: int = Query(default=0, ge=0),
    repo: AsyncProjectRepository = Depends(get_project_repository),
) -> ProjectListResponse:
    """List all projects."""
    projects = await repo.list_projects(limit=limit, offset=offset)
    return ProjectListResponse(
        projects=[ProjectResponse.model_validate(p) for p in projects],
        total=len(projects),
    )


@router.post("", response_model=ProjectResponse, status_code=status.HTTP_201_CREATED)
async def create_project(
    request: ProjectCreate,
    repo: AsyncProjectRepository = Depends(get_project_repository),
) -> ProjectResponse:
    """Create a new project."""
    now = datetime.now(timezone.utc)
    project = Project(
        id=Project.new_id(),
        name=request.name,
        output_width=request.output_width,
        output_height=request.output_height,
        output_fps=request.output_fps,
        created_at=now,
        updated_at=now,
    )
    await repo.add(project)
    return ProjectResponse.model_validate(project)


@router.get("/{project_id}", response_model=ProjectResponse)
async def get_project(
    project_id: str,
    repo: AsyncProjectRepository = Depends(get_project_repository),
) -> ProjectResponse:
    """Get project by ID."""
    project = await repo.get(project_id)
    if project is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Project {project_id} not found"},
        )
    return ProjectResponse.model_validate(project)


@router.delete("/{project_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_project(
    project_id: str,
    repo: AsyncProjectRepository = Depends(get_project_repository),
) -> Response:
    """Delete project."""
    deleted = await repo.delete(project_id)
    if not deleted:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Project {project_id} not found"},
        )
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@router.get("/{project_id}/clips", response_model=ClipListResponse)
async def list_clips(
    project_id: str,
    project_repo: AsyncProjectRepository = Depends(get_project_repository),
    clip_repo: AsyncClipRepository = Depends(get_clip_repository),
) -> ClipListResponse:
    """List clips in project."""
    project = await project_repo.get(project_id)
    if project is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Project {project_id} not found"},
        )
    
    clips = await clip_repo.list_by_project(project_id)
    return ClipListResponse(
        clips=[ClipResponse.model_validate(c) for c in clips],
        total=len(clips),
    )


@router.post("/{project_id}/clips", response_model=ClipResponse, status_code=status.HTTP_201_CREATED)
async def add_clip(
    project_id: str,
    request: ClipCreate,
    project_repo: AsyncProjectRepository = Depends(get_project_repository),
    clip_repo: AsyncClipRepository = Depends(get_clip_repository),
) -> ClipResponse:
    """Add clip to project."""
    project = await project_repo.get(project_id)
    if project is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Project {project_id} not found"},
        )
    
    now = datetime.now(timezone.utc)
    clip = Clip(
        id=Clip.new_id(),
        project_id=project_id,
        source_video_id=request.source_video_id,
        in_point=request.in_point,
        out_point=request.out_point,
        timeline_position=request.timeline_position,
        created_at=now,
        updated_at=now,
    )
    
    # Validate using Rust
    try:
        frame_rate = FrameRate(project.output_fps, 1)
        clip.validate(frame_rate)
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail={"code": "VALIDATION_ERROR", "message": str(e)},
        )
    
    await clip_repo.add(clip)
    return ClipResponse.model_validate(clip)


@router.patch("/{project_id}/clips/{clip_id}", response_model=ClipResponse)
async def update_clip(
    project_id: str,
    clip_id: str,
    request: ClipUpdate,
    project_repo: AsyncProjectRepository = Depends(get_project_repository),
    clip_repo: AsyncClipRepository = Depends(get_clip_repository),
) -> ClipResponse:
    """Update clip."""
    project = await project_repo.get(project_id)
    if project is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Project {project_id} not found"},
        )
    
    clip = await clip_repo.get(clip_id)
    if clip is None or clip.project_id != project_id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Clip {clip_id} not found"},
        )
    
    # Apply updates
    if request.in_point is not None:
        clip.in_point = request.in_point
    if request.out_point is not None:
        clip.out_point = request.out_point
    if request.timeline_position is not None:
        clip.timeline_position = request.timeline_position
    clip.updated_at = datetime.now(timezone.utc)
    
    # Validate using Rust
    try:
        frame_rate = FrameRate(project.output_fps, 1)
        clip.validate(frame_rate)
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail={"code": "VALIDATION_ERROR", "message": str(e)},
        )
    
    await clip_repo.update(clip)
    return ClipResponse.model_validate(clip)


@router.delete("/{project_id}/clips/{clip_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_clip(
    project_id: str,
    clip_id: str,
    clip_repo: AsyncClipRepository = Depends(get_clip_repository),
) -> Response:
    """Delete clip."""
    clip = await clip_repo.get(clip_id)
    if clip is None or clip.project_id != project_id:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={"code": "NOT_FOUND", "message": f"Clip {clip_id} not found"},
        )
    
    await clip_repo.delete(clip_id)
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

## Step 3: Register Router
Update `src/stoat_ferret/api/app.py`:

```python
from stoat_ferret.api.routers import health, videos, projects

def create_app() -> FastAPI:
    # ... existing
    app.include_router(projects.router)
    return app
```

## Step 4: Update Test Fixtures
Update `tests/test_api/conftest.py` to include project and clip repository overrides:

```python
"""Shared fixtures for API tests."""

from collections.abc import Generator
from typing import Any

import pytest
from fastapi import FastAPI
from fastapi.testclient import TestClient

from stoat_ferret.api.app import create_app
from stoat_ferret.api.routers.videos import get_repository as get_video_repository
from stoat_ferret.api.routers.projects import get_project_repository, get_clip_repository
from stoat_ferret.db.async_repository import AsyncInMemoryVideoRepository
from stoat_ferret.db.project_repository import AsyncInMemoryProjectRepository
from stoat_ferret.db.clip_repository import AsyncInMemoryClipRepository


@pytest.fixture
def app() -> FastAPI:
    """Create test application."""
    return create_app()


@pytest.fixture
def video_repository() -> AsyncInMemoryVideoRepository:
    """In-memory video repository for testing."""
    return AsyncInMemoryVideoRepository()


@pytest.fixture
def project_repository() -> AsyncInMemoryProjectRepository:
    """In-memory project repository for testing."""
    return AsyncInMemoryProjectRepository()


@pytest.fixture
def clip_repository() -> AsyncInMemoryClipRepository:
    """In-memory clip repository for testing."""
    return AsyncInMemoryClipRepository()


@pytest.fixture
def client(
    app: FastAPI,
    video_repository: AsyncInMemoryVideoRepository,
    project_repository: AsyncInMemoryProjectRepository,
    clip_repository: AsyncInMemoryClipRepository,
) -> Generator[TestClient, None, None]:
    """Test client with dependency overrides."""
    app.dependency_overrides[get_video_repository] = lambda: video_repository
    app.dependency_overrides[get_project_repository] = lambda: project_repository
    app.dependency_overrides[get_clip_repository] = lambda: clip_repository
    
    with TestClient(app) as c:
        yield c
    
    app.dependency_overrides.clear()
```

## Step 5: Add Tests
Create `tests/test_api/test_projects.py`:

```python
"""Tests for project endpoints."""

import pytest
from fastapi.testclient import TestClient

from stoat_ferret.db.project_repository import AsyncInMemoryProjectRepository


@pytest.mark.api
def test_list_projects_empty(client: TestClient):
    """List returns empty when no projects."""
    response = client.get("/api/v1/projects")
    assert response.status_code == 200
    assert response.json()["projects"] == []


@pytest.mark.api
def test_create_project(client: TestClient):
    """Create project returns 201."""
    response = client.post(
        "/api/v1/projects",
        json={"name": "My Project"},
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "My Project"
    assert data["output_width"] == 1920
    assert data["output_height"] == 1080
    assert data["output_fps"] == 30


@pytest.mark.api
def test_get_project_not_found(client: TestClient):
    """Get returns 404 for unknown ID."""
    response = client.get("/api/v1/projects/nonexistent")
    assert response.status_code == 404


@pytest.mark.api
def test_delete_project_not_found(client: TestClient):
    """Delete returns 404 for unknown ID."""
    response = client.delete("/api/v1/projects/nonexistent")
    assert response.status_code == 404
```

Create `tests/test_api/test_clips.py`:

```python
"""Tests for clip endpoints."""

from datetime import datetime, timezone

import pytest
from fastapi.testclient import TestClient

from stoat_ferret.db.models import Project
from stoat_ferret.db.project_repository import AsyncInMemoryProjectRepository
from stoat_ferret.db.async_repository import AsyncInMemoryVideoRepository
from tests.test_repository_contract import make_test_video


@pytest.mark.api
async def test_list_clips_empty(
    client: TestClient,
    project_repository: AsyncInMemoryProjectRepository,
):
    """List returns empty when no clips."""
    now = datetime.now(timezone.utc)
    project = Project(
        id="proj-1",
        name="Test",
        output_width=1920,
        output_height=1080,
        output_fps=30,
        created_at=now,
        updated_at=now,
    )
    await project_repository.add(project)
    
    response = client.get("/api/v1/projects/proj-1/clips")
    assert response.status_code == 200
    assert response.json()["clips"] == []


@pytest.mark.api
def test_list_clips_project_not_found(client: TestClient):
    """List clips returns 404 for unknown project."""
    response = client.get("/api/v1/projects/nonexistent/clips")
    assert response.status_code == 404


@pytest.mark.api
async def test_add_clip(
    client: TestClient,
    project_repository: AsyncInMemoryProjectRepository,
    video_repository: AsyncInMemoryVideoRepository,
):
    """Add clip returns 201."""
    now = datetime.now(timezone.utc)
    project = Project(
        id="proj-1",
        name="Test",
        output_width=1920,
        output_height=1080,
        output_fps=30,
        created_at=now,
        updated_at=now,
    )
    await project_repository.add(project)
    
    video = make_test_video()
    await video_repository.add(video)
    
    response = client.post(
        "/api/v1/projects/proj-1/clips",
        json={
            "source_video_id": video.id,
            "in_point": 0,
            "out_point": 100,
            "timeline_position": 0,
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["project_id"] == "proj-1"
    assert data["in_point"] == 0
    assert data["out_point"] == 100


@pytest.mark.api
def test_delete_clip_not_found(client: TestClient):
    """Delete clip returns 404 for unknown clip."""
    response = client.delete("/api/v1/projects/proj-1/clips/nonexistent")
    assert response.status_code == 404
```

## Verification
- Project CRUD works
- Clip CRUD works
- Invalid clips rejected with validation error
- `uv run pytest tests/test_api/ -m api` passes
