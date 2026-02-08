You are performing Task 007: Document Drafts for stoat-and-ferret version v004.

Read AGENTS.md first and follow all instructions there.

PROJECT=stoat-and-ferret
VERSION=v004

## Objective

Draft the complete content for all design documents: VERSION_DESIGN.md, THEME_INDEX.md, THEME_DESIGN.md per theme, and requirements.md + implementation-plan.md per feature. Documents must be lean and reference the design artifact store.

## Context

This is Phase 3 (Document Drafts and Persistence) for stoat-and-ferret version v004. The refined logical design from Phase 2 (Task 006) is complete.

WARNING: Do NOT modify any files in comms/outbox/versions/design/v004/. These are the reference artifacts.

## Tasks

### 1. Read the Design Artifact Store

Read ALL outputs from the centralized store:
- `comms/outbox/versions/design/v004/001-environment/README.md`
- `comms/outbox/versions/design/v004/002-backlog/backlog-details.md`
- `comms/outbox/versions/design/v004/002-backlog/README.md`
- `comms/outbox/versions/design/v004/004-research/codebase-patterns.md`
- `comms/outbox/versions/design/v004/004-research/external-research.md`
- `comms/outbox/versions/design/v004/005-logical-design/logical-design.md`
- `comms/outbox/versions/design/v004/005-logical-design/test-strategy.md`
- `comms/outbox/versions/design/v004/006-critical-thinking/refined-logical-design.md`
- `comms/outbox/versions/design/v004/006-critical-thinking/risk-assessment.md`

Use Task 006's refined-logical-design.md as the PRIMARY design source.

### 2. Draft VERSION_DESIGN.md

Keep lean — reference the artifact store for details.

### 3. Draft THEME_INDEX.md

CRITICAL: Machine-Parseable Format. Parser regex: `- (\d+)-([\w-]+):`

REQUIRED format for feature lists:
```
**Features:**

- 001-feature-name: Brief description text here
- 002-another-feature: Another description text
```

FORBIDDEN formats: numbered lists (1. 2. 3.), bold feature identifiers, metadata before colon, multi-line entries, missing colon.

The v004 design has 5 themes and 15 features:

Theme 01: test-foundation (3 features: inmemory-test-doubles, dependency-injection, fixture-factory)
Theme 02: blackbox-contract (3 features: blackbox-test-catalog, ffmpeg-contract-tests, search-unification)
Theme 03: async-scan (3 features: job-queue-infrastructure, async-scan-endpoint, scan-doc-updates)
Theme 04: security-performance (2 features: security-audit, performance-benchmark)
Theme 05: devex-coverage (4 features: property-test-guidance, rust-coverage, coverage-gap-fixes, docker-testing)

### 4. Draft THEME_DESIGN.md (per theme)

For EACH theme, create a lean THEME_DESIGN.md with Goal, Design Artifacts reference, Features table, Dependencies, Technical Approach, and Risks.

### 5. Draft requirements.md (per feature)

For EACH feature: Goal, Background, Functional Requirements, Non-Functional Requirements, Out of Scope, Test Requirements. CRITICAL: Cross-reference BL numbers against the backlog analysis (Task 002). Do NOT write BL numbers from memory.

### 6. Draft implementation-plan.md (per feature)

For EACH feature: Overview, Files to Create/Modify, Implementation Stages, Test Infrastructure Updates, Quality Gates, Risks, Commit Message.

## Output Requirements

Create in comms/outbox/exploration/design-v004-007-drafts/:

### README.md (required)

First paragraph: Summary of document drafts created (X themes, Y features).

### drafts/ folder (structured output)

1. Create `drafts/manifest.json` with version metadata
2. Write `drafts/VERSION_DESIGN.md`
3. Write `drafts/THEME_INDEX.md`
4. For each theme: create `drafts/{theme-slug}/THEME_DESIGN.md`
5. For each feature: create `drafts/{theme-slug}/{feature-slug}/requirements.md` and `drafts/{theme-slug}/{feature-slug}/implementation-plan.md`

#### manifest.json schema

```json
{
  "version": "v004",
  "description": "Testing Infrastructure & Quality Verification",
  "backlog_ids": ["BL-020", "BL-021", "BL-022", "BL-023", "BL-024", "BL-025", "BL-026", "BL-027", "BL-009", "BL-010", "BL-012", "BL-014", "BL-016"],
  "context": {
    "rationale": "...",
    "constraints": ["..."],
    "assumptions": ["..."],
    "deferred_items": []
  },
  "themes": [
    {
      "number": 1,
      "slug": "test-foundation",
      "goal": "...",
      "features": [
        {"number": 1, "slug": "inmemory-test-doubles", "goal": "..."},
        {"number": 2, "slug": "dependency-injection", "goal": "..."},
        {"number": 3, "slug": "fixture-factory", "goal": "..."}
      ]
    },
    ...
  ]
}
```

CRITICAL — Slug Naming:
- Theme slugs must NOT include number prefixes (use "test-foundation", not "01-test-foundation")
- Feature slugs must NOT include number prefixes (use "inmemory-test-doubles", not "010-inmemory-test-doubles")
- The MCP tools add number prefixes automatically

#### Folder layout

```
comms/outbox/exploration/design-v004-007-drafts/
├── README.md
├── draft-checklist.md
└── drafts/
    ├── manifest.json
    ├── VERSION_DESIGN.md
    ├── THEME_INDEX.md
    ├── test-foundation/
    │   ├── THEME_DESIGN.md
    │   ├── inmemory-test-doubles/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   ├── dependency-injection/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   └── fixture-factory/
    │       ├── requirements.md
    │       └── implementation-plan.md
    ├── blackbox-contract/
    │   ├── THEME_DESIGN.md
    │   ├── blackbox-test-catalog/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   ├── ffmpeg-contract-tests/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   └── search-unification/
    │       ├── requirements.md
    │       └── implementation-plan.md
    ├── async-scan/
    │   ├── THEME_DESIGN.md
    │   ├── job-queue-infrastructure/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   ├── async-scan-endpoint/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   └── scan-doc-updates/
    │       ├── requirements.md
    │       └── implementation-plan.md
    ├── security-performance/
    │   ├── THEME_DESIGN.md
    │   ├── security-audit/
    │   │   ├── requirements.md
    │   │   └── implementation-plan.md
    │   └── performance-benchmark/
    │       ├── requirements.md
    │       └── implementation-plan.md
    └── devex-coverage/
        ├── THEME_DESIGN.md
        ├── property-test-guidance/
        │   ├── requirements.md
        │   └── implementation-plan.md
        ├── rust-coverage/
        │   ├── requirements.md
        │   └── implementation-plan.md
        ├── coverage-gap-fixes/
        │   ├── requirements.md
        │   └── implementation-plan.md
        └── docker-testing/
            ├── requirements.md
            └── implementation-plan.md
```

### draft-checklist.md

Verification checklist:
- [ ] manifest.json is valid JSON with all required fields
- [ ] Every theme in manifest has a corresponding folder under drafts/
- [ ] Every feature has requirements.md and implementation-plan.md
- [ ] VERSION_DESIGN.md and THEME_INDEX.md exist
- [ ] THEME_INDEX feature lines match format `- \d{3}-[\w-]+: .+`
- [ ] No placeholder text
- [ ] All backlog IDs from manifest appear in at least one requirements.md
- [ ] No theme or feature slug starts with a digit prefix
- [ ] Backlog IDs cross-referenced against Task 002

## Guidelines

- ALL backlog items from PLAN.md are MANDATORY
- Documents must be LEAN — reference the artifact store, don't duplicate it
- Follow machine-parseable format for THEME_INDEX.md
- Do NOT modify the design artifact store

## When Complete

git add comms/outbox/exploration/design-v004-007-drafts/
git commit -m "exploration: design-v004-007-drafts - document drafts complete"
