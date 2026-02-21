Explore the stoat-and-ferret codebase to understand the current state of logging and debug output:

1. How is logging configured? Look at `src/stoat_ferret/logging.py` and anywhere `configure_logging` is called.
2. Where does log output actually go? Is it stdout-only, or are log files written to disk anywhere? Check for any FileHandler, RotatingFileHandler, or file-based log sinks.
3. Is `configure_logging()` actually called at application startup? Check `app.py`, `main.py`, `__init__.py`, any CLI entry points, and `pyproject.toml` scripts.
4. What log level is used by default? Can it be configured via environment variables or settings?
5. Are there any other logging mechanisms (e.g. Rust-side logging via `tracing` or `log` crate, uvicorn's own logging)?
6. Is there any log rotation, log directory configuration, or persistent debug logging?

Look at the actual implementation, not just design docs. Check `src/`, `rust/`, config files, Docker files, and entry points.

## Output Requirements

Create findings in comms/outbox/exploration/check-logging-output/:

### README.md (required)
First paragraph: Direct answer to "does the MVP log debug output to files anywhere?"
Then: Summary of logging configuration, where output goes, and any gaps.

### logging-config.md
- How logging is configured (structlog setup, handlers, formatters)
- Where configure_logging is called (or if it isn't)
- Default log levels and configurability
- Whether debug logs are emitted anywhere

### log-destinations.md
- All log output destinations (stdout, files, etc.)
- Any file-based logging or log rotation
- Rust-side logging configuration
- uvicorn/ASGI server logging behaviour

## Guidelines
- Under 200 lines per document
- Include actual code snippets from the codebase (not design docs)
- Be specific about what IS implemented vs what is only in design docs
- Commit when complete

## When Complete
git add comms/outbox/exploration/check-logging-output/
git commit -m "exploration: check-logging-output complete"
