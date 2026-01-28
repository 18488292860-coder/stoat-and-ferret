# Implementation Plan: Clip Data Model

## Step 1: Add Clip Model
Update `src/stoat_ferret/db/models.py`:

```python
import uuid
from dataclasses import dataclass
from datetime import datetime

from stoat_ferret_core import Clip as RustClip, ClipValidationError, FrameRate


@dataclass
class Clip:
    """Video clip within a project."""
    
    id: str
    project_id: str
    source_video_id: str
    in_point: int  # frames
    out_point: int  # frames
    timeline_position: int  # frames
    created_at: datetime
    updated_at: datetime

    @classmethod
    def new_id(cls) -> str:
        return str(uuid.uuid4())

    def validate(self, frame_rate: FrameRate) -> None:
        """Validate clip using Rust core.
        
        Raises:
            ClipValidationError: If clip is invalid.
        """
        media_duration = self.out_point - self.in_point
        # Create Rust Clip to validate - will raise ClipValidationError if invalid
        RustClip.new(
            start=self.timeline_position,
            end=self.timeline_position + media_duration,
            media_start=self.in_point,
            media_duration=media_duration,
            frame_rate=frame_rate,
        )
```

## Step 2: Create Migration
```bash
uv run alembic revision -m "add_clips_table"
```

Edit migration file:
```python
"""add_clips_table

Revision ID: <generated>
Revises: <previous - should be add_projects_table>
Create Date: <generated>
"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision: str = '<generated>'
down_revision: Union[str, None] = '<add_projects_table revision>'
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    op.create_table(
        "clips",
        sa.Column("id", sa.Text(), primary_key=True),
        sa.Column("project_id", sa.Text(), sa.ForeignKey("projects.id", ondelete="CASCADE"), nullable=False),
        sa.Column("source_video_id", sa.Text(), sa.ForeignKey("videos.id", ondelete="RESTRICT"), nullable=False),
        sa.Column("in_point", sa.Integer(), nullable=False),
        sa.Column("out_point", sa.Integer(), nullable=False),
        sa.Column("timeline_position", sa.Integer(), nullable=False),
        sa.Column("created_at", sa.Text(), nullable=False),
        sa.Column("updated_at", sa.Text(), nullable=False),
    )
    # Index for efficient project queries
    op.create_index("idx_clips_project", "clips", ["project_id"])
    # Index for timeline ordering within project
    op.create_index("idx_clips_timeline", "clips", ["project_id", "timeline_position"])


def downgrade() -> None:
    op.drop_index("idx_clips_timeline")
    op.drop_index("idx_clips_project")
    op.drop_table("clips")
```

## Step 3: Update Schema Module
Update `src/stoat_ferret/db/schema.py` to include clips table in `create_tables()`:

```python
def create_tables(conn: sqlite3.Connection) -> None:
    """Create all database tables."""
    # ... existing video tables ...
    
    # Projects table
    conn.execute("""
        CREATE TABLE IF NOT EXISTS projects (
            id TEXT PRIMARY KEY,
            name TEXT NOT NULL,
            output_width INTEGER NOT NULL DEFAULT 1920,
            output_height INTEGER NOT NULL DEFAULT 1080,
            output_fps INTEGER NOT NULL DEFAULT 30,
            created_at TEXT NOT NULL,
            updated_at TEXT NOT NULL
        )
    """)
    
    # Clips table
    conn.execute("""
        CREATE TABLE IF NOT EXISTS clips (
            id TEXT PRIMARY KEY,
            project_id TEXT NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
            source_video_id TEXT NOT NULL REFERENCES videos(id) ON DELETE RESTRICT,
            in_point INTEGER NOT NULL,
            out_point INTEGER NOT NULL,
            timeline_position INTEGER NOT NULL,
            created_at TEXT NOT NULL,
            updated_at TEXT NOT NULL
        )
    """)
    conn.execute("CREATE INDEX IF NOT EXISTS idx_clips_project ON clips(project_id)")
    conn.execute("CREATE INDEX IF NOT EXISTS idx_clips_timeline ON clips(project_id, timeline_position)")
    
    conn.commit()
```

## Step 4: Add Tests
Create `tests/test_clip_model.py`:

```python
"""Tests for Clip model validation."""

from datetime import datetime, timezone

import pytest

from stoat_ferret.db.models import Clip
from stoat_ferret_core import ClipValidationError, FrameRate


def make_clip(**kwargs) -> Clip:
    """Create a clip with default values."""
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


class TestClipValidation:
    """Tests for Clip.validate() method."""

    def test_valid_clip_passes_validation(self) -> None:
        """Valid clip passes Rust validation."""
        clip = make_clip(
            in_point=0,
            out_point=100,
            timeline_position=0,
        )
        frame_rate = FrameRate(24, 1)
        
        # Should not raise
        clip.validate(frame_rate)

    def test_valid_clip_with_offset_timeline(self) -> None:
        """Clip with non-zero timeline position passes validation."""
        clip = make_clip(
            in_point=50,
            out_point=150,
            timeline_position=200,
        )
        frame_rate = FrameRate(30, 1)
        
        # Should not raise
        clip.validate(frame_rate)

    def test_invalid_clip_out_before_in_raises(self) -> None:
        """Clip with out_point before in_point raises ClipValidationError."""
        clip = make_clip(
            in_point=100,
            out_point=50,  # Invalid: out < in
            timeline_position=0,
        )
        frame_rate = FrameRate(24, 1)
        
        with pytest.raises(ClipValidationError):
            clip.validate(frame_rate)

    def test_invalid_clip_zero_duration_raises(self) -> None:
        """Clip with zero duration (in == out) raises ClipValidationError."""
        clip = make_clip(
            in_point=100,
            out_point=100,  # Invalid: zero duration
            timeline_position=0,
        )
        frame_rate = FrameRate(24, 1)
        
        with pytest.raises(ClipValidationError):
            clip.validate(frame_rate)

    def test_invalid_clip_negative_in_point_raises(self) -> None:
        """Clip with negative in_point raises ClipValidationError."""
        clip = make_clip(
            in_point=-10,  # Invalid: negative
            out_point=100,
            timeline_position=0,
        )
        frame_rate = FrameRate(24, 1)
        
        with pytest.raises(ClipValidationError):
            clip.validate(frame_rate)

    def test_invalid_clip_negative_timeline_position_raises(self) -> None:
        """Clip with negative timeline_position raises ClipValidationError."""
        clip = make_clip(
            in_point=0,
            out_point=100,
            timeline_position=-50,  # Invalid: negative
        )
        frame_rate = FrameRate(24, 1)
        
        with pytest.raises(ClipValidationError):
            clip.validate(frame_rate)

    def test_different_frame_rates(self) -> None:
        """Validation works with different frame rates."""
        clip = make_clip(
            in_point=0,
            out_point=1000,
            timeline_position=0,
        )
        
        # All should pass
        clip.validate(FrameRate(24, 1))
        clip.validate(FrameRate(30, 1))
        clip.validate(FrameRate(60, 1))
        clip.validate(FrameRate(24000, 1001))  # 23.976 fps


class TestClipNewId:
    """Tests for Clip.new_id() class method."""

    def test_new_id_returns_uuid(self) -> None:
        """new_id returns a valid UUID string."""
        id1 = Clip.new_id()
        id2 = Clip.new_id()
        
        # Should be UUID format (36 chars with hyphens)
        assert len(id1) == 36
        assert id1.count("-") == 4
        
        # Should be unique
        assert id1 != id2
```

## Verification
- Clip model validates via Rust core
- Invalid clips raise ClipValidationError with appropriate message
- `uv run pytest tests/test_clip_model.py -v` passes
- Migration applies cleanly: `uv run alembic upgrade head`
- Migration reverts cleanly: `uv run alembic downgrade -1`
