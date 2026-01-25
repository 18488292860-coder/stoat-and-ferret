# Version Close Process

## Purpose

Finalize the version and prepare for next.

## Inputs

- RETROSPECTIVE.md - Review complete
- All requirements verified

## Process

### 1. Verify Requirements Complete

Review REQUIREMENTS.md:
- All acceptance criteria met
- No outstanding issues
- Documentation updated

### 2. Update ROADMAP.md

Mark version complete:
```markdown
## v{version} - {Title}
**Status:** Complete
**Completed:** {date}
```

### 3. Update BACKLOG.md

- Remove completed items
- Adjust priorities based on learnings
- Add new items discovered

### 4. Tag Version

```bash
git tag v{version}
git push origin v{version}
```

### 5. Create Release Notes

Summarize for stakeholders:
- Key features delivered
- Breaking changes
- Migration notes

### 6. Plan Next Version

Review:
- Remaining backlog items
- Action items from retrospective
- New priorities

## Verification Checklist

- [ ] All requirements verified
- [ ] ROADMAP.md updated
- [ ] BACKLOG.md updated
- [ ] Version tagged
- [ ] Release notes created

## Outputs

- Version tagged and released
- ROADMAP.md shows complete
- Ready for next version

## Next Step

Return to [01-VERSION-SCOPING.md](./01-VERSION-SCOPING.md) for next version
