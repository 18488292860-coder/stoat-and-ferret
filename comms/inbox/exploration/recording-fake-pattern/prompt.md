Research recording fake patterns for testing external process execution (like FFmpeg) to answer:

1. How do recording test doubles capture subprocess commands for verification?
2. What's the contract test pattern - verifying fakes match real implementations?
3. How do other projects test FFmpeg integrations without running FFmpeg in every test?
4. How does this integrate with pytest fixtures and dependency injection?
5. What patterns exist for recording/playback of external command execution?

## Research Approach

Since stoat-and-ferret has no code yet, research external sources:
- pytest-recording and similar libraries
- VCR.py pattern (adapted for subprocess)
- Open source video processing projects (moviepy, ffmpeg-python)
- Testing patterns from auto-dev-mcp (RecordingBackend pattern)

## Output Requirements

Create findings in comms/outbox/exploration/recording-fake-pattern/:

### README.md (required)
First paragraph: Concise summary of recommended approach for stoat-and-ferret.
Then: Overview and links to detailed documents.

### executor-interface.md
Protocol/interface design for FFmpegExecutor with recording capability.

### recording-mechanism.md
How to capture commands, store them, and replay for verification.

### contract-tests.md
Pattern for periodic verification against real FFmpeg.

### pytest-integration.md
Fixtures, conftest.py patterns, and test organization.

## Guidelines
- Under 200 lines per document
- Use clear headings
- Include concrete code snippets showing the pattern
- Show both recording and verification sides

## When Complete
git add comms/outbox/exploration/recording-fake-pattern/
git commit -m "exploration: recording-fake-pattern complete"