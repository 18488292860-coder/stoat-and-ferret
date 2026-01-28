# Implementation Plan: Clip Repository

## Step 1: Create Repository
Create `src/stoat_ferret/db/clip_repository.py`:

```python
"""Clip repository implementations."""

from datetime import datetime
from typing import Protocol

import aiosqlite

from stoat_ferret.db.models import Clip


class AsyncClipRepository(Protocol):
    """Protocol for async clip repository operations."""
    
    async def add(self, clip: Clip) -> Clip: ...
    async def get(self, id: str) -> Clip | None: ...
    async def list_by_project(self, project_id: str) -> list[Clip]: ...
    async def update(self, clip: Clip) -> Clip: ...
    async def delete(self, id: str) -> bool: ...


class AsyncSQLiteClipRepository:
    """Async SQLite implementation of ClipRepository."""
    
    def __init__(self, conn: aiosqlite.Connection) -> None:
        self._conn = conn
        self._conn.row_factory = aiosqlite.Row

    async def add(self, clip: Clip) -> Clip:
        """Add a clip to the repository."""
        try:
            await self._conn.execute(
                """
                INSERT INTO clips (id, project_id, source_video_id, in_point, out_point,
                                  timeline_position, created_at, updated_at)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                """,
                (clip.id, clip.project_id, clip.source_video_id, clip.in_point,
                 clip.out_point, clip.timeline_position,
                 clip.created_at.isoformat(), clip.updated_at.isoformat()),
            )
            await self._conn.commit()
        except aiosqlite.IntegrityError as e:
            raise ValueError(f"Clip already exists or foreign key violation: {e}") from e
        return clip

    async def get(self, id: str) -> Clip | None:
        """Get a clip by ID."""
        cursor = await self._conn.execute("SELECT * FROM clips WHERE id = ?", (id,))
        row = await cursor.fetchone()
        return self._row_to_clip(row) if row else None

    async def list_by_project(self, project_id: str) -> list[Clip]:
        """List clips in a project, ordered by timeline position."""
        cursor = await self._conn.execute(
            "SELECT * FROM clips WHERE project_id = ? ORDER BY timeline_position",
            (project_id,),
        )
        rows = await cursor.fetchall()
        return [self._row_to_clip(row) for row in rows]

    async def update(self, clip: Clip) -> Clip:
        """Update an existing clip."""
        cursor = await self._conn.execute(
            """
            UPDATE clips SET
                in_point = ?, out_point = ?, timeline_position = ?, updated_at = ?
            WHERE id = ?
            """,
            (clip.in_point, clip.out_point, clip.timeline_position,
             clip.updated_at.isoformat(), clip.id),
        )
        await self._conn.commit()
        if cursor.rowcount == 0:
            raise ValueError(f"Clip {clip.id} does not exist")
        return clip

    async def delete(self, id: str) -> bool:
        """Delete a clip by ID."""
        cursor = await self._conn.execute("DELETE FROM clips WHERE id = ?", (id,))
        await self._conn.commit()
        return cursor.rowcount > 0

    def _row_to_clip(self, row: aiosqlite.Row) -> Clip:
        """Convert database row to Clip."""
        return Clip(
            id=row["id"],
            project_id=row["project_id"],
            source_video_id=row["source_video_id"],
            in_point=row["in_point"],
            out_point=row["out_point"],
            timeline_position=row["timeline_position"],
            created_at=datetime.fromisoformat(row["created_at"]),
            updated_at=datetime.fromisoformat(row["updated_at"]),
        )


class AsyncInMemoryClipRepository:
    """Async in-memory implementation for testing."""
    
    def __init__(self) -> None:
        self._clips: dict[str, Clip] = {}

    async def add(self, clip: Clip) -> Clip:
        """Add a clip to the repository."""
        if clip.id in self._clips:
            raise ValueError(f"Clip {clip.id} already exists")
        self._clips[clip.id] = clip
        return clip

    async def get(self, id: str) -> Clip | None:
        """Get a clip by ID."""
        return self._clips.get(id)

    async def list_by_project(self, project_id: str) -> list[Clip]:
        """List clips in a project, ordered by timeline position."""
        clips = [c for c in self._clips.values() if c.project_id == project_id]
        return sorted(clips, key=lambda c: c.timeline_position)

    async def update(self, clip: Clip) -> Clip:
        """Update an existing clip."""
        if clip.id not in self._clips:
            raise ValueError(f"Clip {clip.id} does not exist")
        self._clips[clip.id] = clip
        return clip

    async def delete(self, id: str) -> bool:
        """Delete a clip by ID."""
        if id not in self._clips:
            return False
        del self._clips[id]
        return True
```

## Step 2: Update Module Exports
Update `src/stoat_ferret/db/__init__.py`:
```python
from stoat_ferret.db.clip_repository import (
    AsyncClipRepository,
    AsyncInMemoryClipRepository,
    AsyncSQLiteClipRepository,
)
```

## Step 3: Add Contract Tests
Create `tests/test_clip_repository_contract.py`:

```python
"""Contract tests for async ClipRepository implementations."""

import sqlite3
from collections.abc import AsyncGenerator
from datetime import datetime, timezone

import aiosqlite
import pytest

from stoat_ferret.db.clip_repository import (
    AsyncInMemoryClipRepository,
    AsyncSQLiteClipRepository,
)
from stoat_ferret.db.models import Clip
from stoat_ferret.db.schema import create_tables


def make_test_clip(**kwargs) -> Clip:
    """Create a test clip with default values."""
    now = datetime.now(timezone.utc)
    defaults = {
        "id": Clip.new_id(),
        "project_id": "project-1",
        "source_video_id": "video-1",
        "in_point": 0,
        "out_point": 100,
        "timeline_position": 0,
        "created_at": now,
        "updated_at": now,
    }
    defaults.update(kwargs)
    return Clip(**defaults)


AsyncClipRepositoryType = AsyncSQLiteClipRepository | AsyncInMemoryClipRepository


@pytest.fixture(params=["sqlite", "memory"])
async def clip_repository(request: pytest.FixtureRequest) -> AsyncGenerator[AsyncClipRepositoryType, None]:
    """Provide both clip repository implementations."""
    if request.param == "sqlite":
        # Create schema using sync connection
        sync_conn = sqlite3.connect(":memory:")
        create_tables(sync_conn)
        # Add project and video for foreign keys
        sync_conn.execute(
            """INSERT INTO projects (id, name, output_width, output_height, output_fps, created_at, updated_at)
               VALUES ('project-1', 'Test', 1920, 1080, 30, '2024-01-01T00:00:00', '2024-01-01T00:00:00')"""
        )
        sync_conn.execute(
            """INSERT INTO videos (id, path, filename, duration_frames, frame_rate_numerator, frame_rate_denominator,
               width, height, video_codec, file_size, created_at, updated_at)
               VALUES ('video-1', '/test.mp4', 'test.mp4', 1000, 24, 1, 1920, 1080, 'h264', 1000000,
               '2024-01-01T00:00:00', '2024-01-01T00:00:00')"""
        )
        sync_conn.commit()
        schema_dump = list(sync_conn.iterdump())
        sync_conn.close()
        
        # Apply schema to async connection
        conn = await aiosqlite.connect(":memory:")
        for line in schema_dump:
            await conn.execute(line)
        await conn.commit()
        
        yield AsyncSQLiteClipRepository(conn)
        await conn.close()
    else:
        yield AsyncInMemoryClipRepository()


@pytest.mark.contract
class TestAsyncClipAddAndGet:
    """Tests for async add() and get() methods."""

    async def test_add_and_get(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Adding a clip allows retrieving it by ID."""
        clip = make_test_clip()
        await clip_repository.add(clip)
        retrieved = await clip_repository.get(clip.id)

        assert retrieved is not None
        assert retrieved.id == clip.id
        assert retrieved.project_id == clip.project_id
        assert retrieved.in_point == clip.in_point
        assert retrieved.out_point == clip.out_point

    async def test_get_nonexistent_returns_none(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Getting a nonexistent clip returns None."""
        result = await clip_repository.get("nonexistent-id")
        assert result is None

    async def test_add_duplicate_id_raises(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Adding a clip with duplicate ID raises ValueError."""
        clip = make_test_clip()
        await clip_repository.add(clip)

        duplicate = make_test_clip(id=clip.id, timeline_position=200)
        with pytest.raises(ValueError):
            await clip_repository.add(duplicate)


@pytest.mark.contract
class TestAsyncClipListByProject:
    """Tests for async list_by_project() method."""

    async def test_list_empty_project(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Listing empty project returns empty list."""
        result = await clip_repository.list_by_project("project-1")
        assert result == []

    async def test_list_returns_ordered_clips(self, clip_repository: AsyncClipRepositoryType) -> None:
        """list_by_project returns clips ordered by timeline position."""
        clip1 = make_test_clip(timeline_position=100)
        clip2 = make_test_clip(timeline_position=0)
        clip3 = make_test_clip(timeline_position=50)
        
        await clip_repository.add(clip1)
        await clip_repository.add(clip2)
        await clip_repository.add(clip3)
        
        clips = await clip_repository.list_by_project("project-1")
        assert len(clips) == 3
        assert clips[0].timeline_position == 0
        assert clips[1].timeline_position == 50
        assert clips[2].timeline_position == 100

    async def test_list_filters_by_project(self, clip_repository: AsyncClipRepositoryType) -> None:
        """list_by_project only returns clips for specified project."""
        clip1 = make_test_clip(project_id="project-1")
        clip2 = make_test_clip(project_id="project-2")
        
        await clip_repository.add(clip1)
        await clip_repository.add(clip2)
        
        clips = await clip_repository.list_by_project("project-1")
        assert len(clips) == 1
        assert clips[0].project_id == "project-1"


@pytest.mark.contract
class TestAsyncClipUpdate:
    """Tests for async update() method."""

    async def test_update_existing(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Updating existing clip succeeds."""
        clip = make_test_clip()
        await clip_repository.add(clip)
        
        clip.in_point = 50
        clip.out_point = 150
        clip.timeline_position = 200
        await clip_repository.update(clip)
        
        retrieved = await clip_repository.get(clip.id)
        assert retrieved is not None
        assert retrieved.in_point == 50
        assert retrieved.out_point == 150
        assert retrieved.timeline_position == 200

    async def test_update_nonexistent_raises(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Updating nonexistent clip raises ValueError."""
        clip = make_test_clip()
        with pytest.raises(ValueError):
            await clip_repository.update(clip)


@pytest.mark.contract
class TestAsyncClipDelete:
    """Tests for async delete() method."""

    async def test_delete_existing(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Deleting existing clip returns True."""
        clip = make_test_clip()
        await clip_repository.add(clip)
        
        result = await clip_repository.delete(clip.id)
        assert result is True
        assert await clip_repository.get(clip.id) is None

    async def test_delete_nonexistent(self, clip_repository: AsyncClipRepositoryType) -> None:
        """Deleting nonexistent clip returns False."""
        result = await clip_repository.delete("nonexistent")
        assert result is False
```

## Verification
- All CRUD operations work
- list_by_project returns ordered clips
- Contract tests pass for both implementations
- `uv run pytest tests/test_clip_repository_contract.py -v` passes
