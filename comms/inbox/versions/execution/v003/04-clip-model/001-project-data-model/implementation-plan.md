# Implementation Plan: Project Data Model

## Step 1: Add Project Model
Update `src/stoat_ferret/db/models.py`:

```python
import uuid
from dataclasses import dataclass
from datetime import datetime


@dataclass
class Project:
    """Video editing project."""
    
    id: str
    name: str
    output_width: int
    output_height: int
    output_fps: int
    created_at: datetime
    updated_at: datetime

    @classmethod
    def new_id(cls) -> str:
        return str(uuid.uuid4())
```

## Step 2: Create Migration
```bash
uv run alembic revision -m "add_projects_table"
```

Edit migration:
```python
"""add_projects_table

Revision ID: <generated>
Revises: <previous>
Create Date: <generated>
"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision: str = '<generated>'
down_revision: Union[str, None] = '<previous>'
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    op.create_table(
        "projects",
        sa.Column("id", sa.Text(), primary_key=True),
        sa.Column("name", sa.Text(), nullable=False),
        sa.Column("output_width", sa.Integer(), nullable=False, server_default="1920"),
        sa.Column("output_height", sa.Integer(), nullable=False, server_default="1080"),
        sa.Column("output_fps", sa.Integer(), nullable=False, server_default="30"),
        sa.Column("created_at", sa.Text(), nullable=False),
        sa.Column("updated_at", sa.Text(), nullable=False),
    )


def downgrade() -> None:
    op.drop_table("projects")
```

## Step 3: Create Repository
Create `src/stoat_ferret/db/project_repository.py`:

```python
"""Project repository implementations."""

from datetime import datetime
from typing import Protocol

import aiosqlite

from stoat_ferret.db.models import Project


class AsyncProjectRepository(Protocol):
    """Protocol for async project repository operations."""
    
    async def add(self, project: Project) -> Project: ...
    async def get(self, id: str) -> Project | None: ...
    async def list_projects(self, limit: int = 100, offset: int = 0) -> list[Project]: ...
    async def update(self, project: Project) -> Project: ...
    async def delete(self, id: str) -> bool: ...


class AsyncSQLiteProjectRepository:
    """Async SQLite implementation of ProjectRepository."""
    
    def __init__(self, conn: aiosqlite.Connection) -> None:
        self._conn = conn
        self._conn.row_factory = aiosqlite.Row

    async def add(self, project: Project) -> Project:
        """Add a project to the repository."""
        try:
            await self._conn.execute(
                """
                INSERT INTO projects (id, name, output_width, output_height, output_fps, created_at, updated_at)
                VALUES (?, ?, ?, ?, ?, ?, ?)
                """,
                (project.id, project.name, project.output_width, project.output_height,
                 project.output_fps, project.created_at.isoformat(), project.updated_at.isoformat()),
            )
            await self._conn.commit()
        except aiosqlite.IntegrityError as e:
            raise ValueError(f"Project already exists: {e}") from e
        return project

    async def get(self, id: str) -> Project | None:
        """Get a project by ID."""
        cursor = await self._conn.execute("SELECT * FROM projects WHERE id = ?", (id,))
        row = await cursor.fetchone()
        return self._row_to_project(row) if row else None

    async def list_projects(self, limit: int = 100, offset: int = 0) -> list[Project]:
        """List projects with pagination."""
        cursor = await self._conn.execute(
            "SELECT * FROM projects ORDER BY created_at DESC LIMIT ? OFFSET ?",
            (limit, offset),
        )
        rows = await cursor.fetchall()
        return [self._row_to_project(row) for row in rows]

    async def update(self, project: Project) -> Project:
        """Update an existing project."""
        cursor = await self._conn.execute(
            """
            UPDATE projects SET name = ?, output_width = ?, output_height = ?, 
                output_fps = ?, updated_at = ?
            WHERE id = ?
            """,
            (project.name, project.output_width, project.output_height,
             project.output_fps, project.updated_at.isoformat(), project.id),
        )
        await self._conn.commit()
        if cursor.rowcount == 0:
            raise ValueError(f"Project {project.id} does not exist")
        return project

    async def delete(self, id: str) -> bool:
        """Delete a project by ID."""
        cursor = await self._conn.execute("DELETE FROM projects WHERE id = ?", (id,))
        await self._conn.commit()
        return cursor.rowcount > 0

    def _row_to_project(self, row: aiosqlite.Row) -> Project:
        """Convert database row to Project."""
        return Project(
            id=row["id"],
            name=row["name"],
            output_width=row["output_width"],
            output_height=row["output_height"],
            output_fps=row["output_fps"],
            created_at=datetime.fromisoformat(row["created_at"]),
            updated_at=datetime.fromisoformat(row["updated_at"]),
        )


class AsyncInMemoryProjectRepository:
    """Async in-memory implementation for testing."""
    
    def __init__(self) -> None:
        self._projects: dict[str, Project] = {}

    async def add(self, project: Project) -> Project:
        """Add a project to the repository."""
        if project.id in self._projects:
            raise ValueError(f"Project {project.id} already exists")
        self._projects[project.id] = project
        return project

    async def get(self, id: str) -> Project | None:
        """Get a project by ID."""
        return self._projects.get(id)

    async def list_projects(self, limit: int = 100, offset: int = 0) -> list[Project]:
        """List projects with pagination."""
        sorted_projects = sorted(
            self._projects.values(), key=lambda p: p.created_at, reverse=True
        )
        return sorted_projects[offset : offset + limit]

    async def update(self, project: Project) -> Project:
        """Update an existing project."""
        if project.id not in self._projects:
            raise ValueError(f"Project {project.id} does not exist")
        self._projects[project.id] = project
        return project

    async def delete(self, id: str) -> bool:
        """Delete a project by ID."""
        if id not in self._projects:
            return False
        del self._projects[id]
        return True
```

## Step 4: Update Module Exports
Update `src/stoat_ferret/db/__init__.py`:
```python
from stoat_ferret.db.project_repository import (
    AsyncInMemoryProjectRepository,
    AsyncProjectRepository,
    AsyncSQLiteProjectRepository,
)
```

## Step 5: Add Contract Tests
Create `tests/test_project_repository_contract.py`:

```python
"""Contract tests for async ProjectRepository implementations."""

import sqlite3
from collections.abc import AsyncGenerator
from datetime import datetime, timezone

import aiosqlite
import pytest

from stoat_ferret.db.project_repository import (
    AsyncInMemoryProjectRepository,
    AsyncSQLiteProjectRepository,
)
from stoat_ferret.db.models import Project
from stoat_ferret.db.schema import create_tables


def make_test_project(**kwargs) -> Project:
    """Create a test project with default values."""
    now = datetime.now(timezone.utc)
    defaults = {
        "id": Project.new_id(),
        "name": "Test Project",
        "output_width": 1920,
        "output_height": 1080,
        "output_fps": 30,
        "created_at": now,
        "updated_at": now,
    }
    defaults.update(kwargs)
    return Project(**defaults)


AsyncProjectRepositoryType = AsyncSQLiteProjectRepository | AsyncInMemoryProjectRepository


@pytest.fixture(params=["sqlite", "memory"])
async def project_repository(request: pytest.FixtureRequest) -> AsyncGenerator[AsyncProjectRepositoryType, None]:
    """Provide both project repository implementations."""
    if request.param == "sqlite":
        sync_conn = sqlite3.connect(":memory:")
        create_tables(sync_conn)
        schema_dump = list(sync_conn.iterdump())
        sync_conn.close()
        
        conn = await aiosqlite.connect(":memory:")
        for line in schema_dump:
            if not line.startswith("INSERT"):
                await conn.execute(line)
        await conn.commit()
        
        yield AsyncSQLiteProjectRepository(conn)
        await conn.close()
    else:
        yield AsyncInMemoryProjectRepository()


@pytest.mark.contract
class TestAsyncProjectAddAndGet:
    """Tests for async add() and get() methods."""

    async def test_add_and_get(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Adding a project allows retrieving it by ID."""
        project = make_test_project()
        await project_repository.add(project)
        retrieved = await project_repository.get(project.id)

        assert retrieved is not None
        assert retrieved.id == project.id
        assert retrieved.name == project.name

    async def test_get_nonexistent_returns_none(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Getting a nonexistent project returns None."""
        result = await project_repository.get("nonexistent-id")
        assert result is None

    async def test_add_duplicate_id_raises(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Adding a project with duplicate ID raises ValueError."""
        project = make_test_project()
        await project_repository.add(project)

        duplicate = make_test_project(id=project.id, name="Different Name")
        with pytest.raises(ValueError):
            await project_repository.add(duplicate)


@pytest.mark.contract
class TestAsyncProjectList:
    """Tests for async list_projects() method."""

    async def test_list_empty(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Listing empty repository returns empty list."""
        result = await project_repository.list_projects()
        assert result == []

    async def test_list_with_limit(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Limit restricts number of returned projects."""
        for i in range(5):
            await project_repository.add(make_test_project(name=f"Project {i}"))

        result = await project_repository.list_projects(limit=3)
        assert len(result) == 3


@pytest.mark.contract
class TestAsyncProjectUpdate:
    """Tests for async update() method."""

    async def test_update_existing(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Updating existing project succeeds."""
        project = make_test_project()
        await project_repository.add(project)
        
        project.name = "Updated Name"
        project.output_fps = 60
        await project_repository.update(project)
        
        retrieved = await project_repository.get(project.id)
        assert retrieved is not None
        assert retrieved.name == "Updated Name"
        assert retrieved.output_fps == 60

    async def test_update_nonexistent_raises(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Updating nonexistent project raises ValueError."""
        project = make_test_project()
        with pytest.raises(ValueError):
            await project_repository.update(project)


@pytest.mark.contract
class TestAsyncProjectDelete:
    """Tests for async delete() method."""

    async def test_delete_existing(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Deleting existing project returns True."""
        project = make_test_project()
        await project_repository.add(project)
        
        result = await project_repository.delete(project.id)
        assert result is True
        assert await project_repository.get(project.id) is None

    async def test_delete_nonexistent(self, project_repository: AsyncProjectRepositoryType) -> None:
        """Deleting nonexistent project returns False."""
        result = await project_repository.delete("nonexistent")
        assert result is False
```

## Verification
- Migration applies cleanly
- Repository CRUD works
- `uv run pytest tests/test_project_repository_contract.py -v` passes
- `uv run pytest -m contract` runs all contract tests
