Read AGENTS.md first and follow all instructions there.

## Objective

Call the MCP design tools to persist all drafted documents from Task 007 to the inbox folder structure for stoat-and-ferret version v005.

## Context

This is Phase 3 (Document Drafts & Persistence) for stoat-and-ferret version v005. Task 007 created all document drafts in `comms/outbox/exploration/design-v005-007-drafts/drafts/`. Now persist them using MCP tools.

**WARNING:** Do NOT modify any files in `comms/outbox/versions/design/v005/`. These are the reference artifacts.

## CRITICAL: Machine-Parseable Format Requirements

### THEME_INDEX.md - Feature List Format

**Parser regex:** `- (\d+)-([\w-]+):`

**REQUIRED format:**
```
**Features:**

- 001-feature-name: Brief description text here
- 002-another-feature: Another description text
```

## Tasks

### 1. Read Task 007 Output

Read the manifest and individual draft files from Task 007:

```
drafts_dir = comms/outbox/exploration/design-v005-007-drafts/drafts/
```

1. Read `{drafts_dir}/manifest.json` — contains version metadata, theme/feature numbering, goals, backlog IDs, and context
2. Read `{drafts_dir}/VERSION_DESIGN.md`
3. Read `{drafts_dir}/THEME_INDEX.md`
4. For each theme in manifest: read `{drafts_dir}/{theme_slug}/THEME_DESIGN.md`
5. For each feature in manifest: read `{drafts_dir}/{theme_slug}/{feature_slug}/requirements.md` and `{drafts_dir}/{theme_slug}/{feature_slug}/implementation-plan.md`

**CRITICAL — Use Slugs from Manifest:**
- Pass `theme["slug"]` as the `theme_name` parameter
- Pass `feature["slug"]` as the feature `name` parameter
- Do NOT add number prefixes — the MCP tools add them automatically
- Passing pre-numbered names causes double-numbering bugs

**Error Handling:** If any file referenced by the manifest is missing:
1. List all missing files
2. Report which themes/features are affected
3. STOP without calling any MCP tools

### 2. Prepare Context Object

Read context directly from manifest.json.

### 3. Prepare Themes Array

Build themes structure from manifest. Use slugs (NOT numbered names):

```python
themes = [
    {
        "name": theme["slug"],          # slug only, tool adds prefix
        "goal": theme["goal"],
        "features": [
            {"name": f["slug"], "goal": f["goal"]}
            for f in theme["features"]
        ]
    }
    for theme in manifest["themes"]
]
```

**CRITICAL:** Verify structure:
- themes is a list (not a string)
- Each theme name is a slug WITHOUT number prefix
- Each feature name is a slug WITHOUT number prefix
- Each theme has name, goal, features keys
- Each feature has name, goal keys

### 4. Call design_version

```python
design_version(
    project="stoat-and-ferret",
    version=manifest["version"],
    description=manifest["description"],
    themes=themes,
    backlog_ids=manifest["backlog_ids"],
    context=manifest["context"],
    overwrite=false
)
```

### 5. Call design_theme for Each Theme

For EACH theme, use manifest-driven loop:

```python
for theme in manifest["themes"]:
    theme_design = read_file(f"{drafts_dir}/{theme['slug']}/THEME_DESIGN.md")
    features = []
    for f in theme["features"]:
        req = read_file(f"{drafts_dir}/{theme['slug']}/{f['slug']}/requirements.md")
        plan = read_file(f"{drafts_dir}/{theme['slug']}/{f['slug']}/implementation-plan.md")
        features.append({
            "number": f["number"],
            "name": f["slug"],              # slug only, tool adds prefix
            "requirements": req,
            "implementation_plan": plan      # underscore, not hyphen
        })

    design_theme(
        project="stoat-and-ferret",
        version=manifest["version"],
        theme_number=theme["number"],
        theme_name=theme["slug"],
        theme_design=theme_design,
        features=features,
        mode="full"
    )
```

**CRITICAL - Feature Object Required Fields:**
Each feature dict MUST contain ALL of these fields:
- `number` (int): Feature number within the theme, 1-indexed sequential
- `name` (str): Feature slug WITHOUT number prefix
- `requirements` (str): Full requirements.md markdown content
- `implementation_plan` (str): Full implementation-plan.md markdown content (underscore, NOT hyphen)

### 6. Validate Design Completeness

Call `validate_version_design(project="stoat-and-ferret", version="v005")`.

Expected: All documents exist, no missing files.

## Output Requirements

Create in `comms/outbox/exploration/design-v005-008-persist/`:

### README.md (required)

First paragraph: Summary of persistence operation (success/failure, documents created).

Then:
- **Design Version Call**: Result and any errors
- **Design Theme Calls**: Result for each theme
- **Validation Result**: Output from validate_version_design
- **Missing Documents**: Any documents not created (should be none)

### persistence-log.md

Detailed log of all MCP calls with parameters, results, and any errors.

### verification-checklist.md

Document existence verification for all expected inbox files.

## Allowed MCP Tools

- `read_document`
- `design_version`
- `design_theme`
- `validate_version_design`

## Guidelines

- ALL backlog items from PLAN.md are MANDATORY — verify all items appear in persisted documents
- Verify array structure before calling design_version
- If any MCP call fails, document the error clearly and STOP
- Validate that ALL documents were created successfully
- Do NOT modify the design artifact store

Do NOT commit or push — the calling prompt handles commits. Results folder: design-v005-008-persist.