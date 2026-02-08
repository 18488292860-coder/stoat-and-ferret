# Discrepancies

## Non-Blocking Warnings

### 1. THEME_INDEX Feature Descriptions Are Placeholders

**Location:** `comms/inbox/versions/execution/v004/THEME_INDEX.md`
**Issue:** All 15 feature entries use `_Feature description_` placeholder text instead of actual descriptions.
**Impact:** Cosmetic only. Feature descriptions are fully documented in each feature's `requirements.md`.
**Remediation:** None required. This is standard `design_version` tool behavior.

### 2. Security-Audit Settings Path Discrepancy

**Location:** `comms/inbox/versions/execution/v004/04-security-performance/001-security-audit/implementation-plan.md`, line 13
**Issue:** References `src/stoat_ferret/settings.py` but file is at `src/stoat_ferret/api/settings.py`.
**Impact:** Implementer needs to use correct path. Non-blocking -- the settings file exists.
**Remediation:** Implementer should reference `src/stoat_ferret/api/settings.py` during implementation.

### 3. Async-Scan Models Directory

**Location:** `comms/inbox/versions/execution/v004/03-async-scan/002-async-scan-endpoint/implementation-plan.md`
**Issue:** Creates `src/stoat_ferret/api/models/job.py` but `api/models/` doesn't exist. Codebase uses `api/schemas/` for Pydantic models.
**Impact:** Implementer should decide whether to create `models/` or use existing `schemas/`. Non-blocking.
**Remediation:** Implementer should follow existing codebase conventions (likely `schemas/`).

### 4. CI Workflow Filename

**Location:** `comms/inbox/versions/execution/v004/05-devex-coverage/002-rust-coverage/implementation-plan.md`
**Issue:** References `.github/workflows/test.yml` but actual workflow is `.github/workflows/ci.yml`.
**Impact:** Implementer needs correct filename. Non-blocking.
**Remediation:** Implementer should modify `.github/workflows/ci.yml`.

### 5. Minor Text Differences in search-unification

**Location:** `comms/inbox/versions/execution/v004/02-blackbox-contract/003-search-unification/requirements.md`, line 9
**Issue:** Tiny differences from draft: `'"{}\"*'` vs `'"{query}"*'` and `#2-3` vs `#2-#3`.
**Impact:** Zero functional impact. Background context text only.
**Remediation:** None required.

## Blocking Issues

None identified.
