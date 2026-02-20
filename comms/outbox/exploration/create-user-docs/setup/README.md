# Setup Documentation

This directory contains everything you need to get a working **stoat-and-ferret** development environment. stoat-and-ferret is an AI-driven video editor with a hybrid architecture: a Python/FastAPI HTTP layer, a Rust core compiled via PyO3/maturin, and a React/Vite/Tailwind frontend GUI.

## Documentation Index

| Document | Description |
|----------|-------------|
| [Prerequisites](prerequisites.md) | Required tools and OS-specific installation notes |
| [Development Setup](development-setup.md) | Step-by-step guide to build and run the project locally |
| [Docker Setup](docker-setup.md) | Containerized testing with Docker Compose |
| [Configuration](configuration.md) | Environment variables, settings, and database configuration |
| [Troubleshooting](troubleshooting.md) | Common issues and how to fix them |

## Quick Start

If you already have all prerequisites installed (Python 3.10+, Rust, Node.js 22+, FFmpeg, uv), here is the shortest path to a running dev server:

```bash
# Clone and enter the project
git clone <repo-url> stoat-and-ferret
cd stoat-and-ferret

# Install Python dependencies
uv sync

# Build the Rust core extension
uv run maturin develop

# Install and build the frontend
cd gui && npm ci && npm run build && cd ..

# Initialize the database
uv run alembic upgrade head

# Start the development server
uv run uvicorn stoat_ferret.api.app:create_app --factory --reload
```

The API is then available at `http://127.0.0.1:8000/api/v1/` and the GUI at `http://127.0.0.1:8000/gui/`.

For detailed instructions, start with [Prerequisites](prerequisites.md) and then follow [Development Setup](development-setup.md).
