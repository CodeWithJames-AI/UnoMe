# Task Contract Template

*A contract defines what "done" means. One session per contract. Fresh context for each.*

## Contract: {TASK_NAME}

**Phase**: {phase_number}
**Created**: {date}

### Deliverables
- {specific deliverable 1}
- {specific deliverable 2}

### Acceptance Tests
```bash
# {description of what must pass}
{command 1}
{command 2}
```

### Regression Gate
```bash
# {existing features that must still work}
{regression test command}
```

### Visual Verification (if UI)
- [ ] Screenshot taken and design matches reference
- [ ] No visual regressions in existing features

### Definition of Done
- [ ] All acceptance tests pass (DO NOT edit tests to make them pass)
- [ ] All regression tests pass
- [ ] Visual verification complete (if UI)
- [ ] No orphan files or dead code introduced
- [ ] Docs updated if scope changed
- [ ] Git tag created: `git tag -a unome/{checkpoint} -m "{message}"`

### Do NOT
- Add features beyond what's specified
- Refactor code not touched by this contract
- Create helper utilities for one-time operations
- Leave TODO comments or stubs
