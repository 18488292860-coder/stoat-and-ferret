# Exclusion Log

## Excluded Directories (pattern match)
- `node_modules/` — dependency directory
- `.git/` — version control
- `__pycache__/` — Python bytecode cache
- `.mypy_cache/` — type checker cache
- `.pytest_cache/` — test cache
- `docs/C4-Documentation/` — output directory (excluded by rule)
- `comms/` — auto-dev communication (excluded by rule)
- `docs/auto-dev/` — auto-dev prompts (excluded by rule)
- `docs/ideation/` — ideation docs (excluded by rule)
- `build/`, `dist/` — build artifacts
- `.venv/`, `venv/` — virtual environments
- `gui/node_modules/` — GUI dependency directory

## Excluded Directories (no source code)
- `docs/` — documentation only (markdown)
- `docs/architecture/` — architecture docs (markdown)
- `docs/roadmap/` — roadmap docs (markdown)
- `.github/` — CI/CD configuration (YAML)
- `gui/public/` — static assets only

## Included Directories (36 total)
All directories listed in directory-manifest.md (14 changed + 22 unchanged).
