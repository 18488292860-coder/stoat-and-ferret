## UV.exe Blocked by Windows Smart App Control - Workaround Research

Windows Security is blocking uv.exe when called from Git Bash with message:
"Part of this app has been blocked - Some features of C:\Program Files\Git\usr\bin\bash.exe may not work because we can't confirm who published uv.exe that the app tried to load."

### Key Insight
The block is on uv.exe being loaded by bash.exe, not on Python directly.

### Research Tasks

1. **Test direct CMD execution instead of bash**
   - Run in PowerShell: `uv run pytest tests/ -v`
   - Run in CMD: `uv run pytest tests/ -v`
   - Does the block only happen from Git Bash?

2. **Test py launcher approach**
   - `py -m pytest tests/ -v` (Windows Python launcher)
   - `python -m pytest tests/ -v`
   - These bypass uv entirely

3. **Check if uv.exe can be code-signed**
   - `Get-AuthenticodeSignature (Get-Command uv).Source`
   - Is uv signed? If not, can we use a signed version?

4. **Test alternative package managers**
   - Can we use pip directly? `pip install pytest && pytest tests/`
   - Can we use system Python?

5. **Check Smart App Control settings**
   - Is this SAC or WDAC?
   - Run: `Get-MpComputerStatus | Select-Object SmartAppControlState`
   - Document the current security posture

6. **Research uv alternatives for test execution**
   - Can pyproject.toml define a script that uses python directly?
   - Can we create a test runner script that doesn't use uv?

### Output Requirements

Save findings to: `comms/outbox/exploration/sac-uv-workarounds/`

Create:
- `README.md` - Summary with working solution
- `test-results.md` - Results of each test approach
- `recommended-fix.md` - Specific steps to fix this for auto-dev execution

Commit with message: "docs(exploration): uv.exe SAC blocking workarounds"
