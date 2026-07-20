# Project conventions

Used by the automated PR review and any Claude Code sessions in this repo.

## Code style

- No comments unless the WHY is non-obvious (hidden constraint, subtle invariant, specific bug workaround)
- No dead code, debug artefacts, or backwards-compat shims for removed code
- No secrets or credentials hardcoded — use environment variables or a secrets manager

## Testing language

- "Tested" means a test runner (pytest, jest, cargo test, etc.) was actually executed and stdout is attached
- "Tested" does NOT mean typecheck, lint, or compile-only commands
- If you claim "N/N tests pass", you must have run the command this session

## PR standards

- One issue → one branch → one PR
- Branch name: `{prefix}/{issue-id}-{slug}` (e.g. `feat/APO-3-auth-flow`)
- Spec must be filled before implementation starts
- Self-review checklist signed off before promoting from draft to ready-for-review
- No force-pushes to main; rebase or merge only via PR

## Self-review checklist (run before promoting PR)

### Correctness
- [ ] Each AC from the spec is met and evidenced by test output
- [ ] No regression in existing tests (full suite output attached)

### Code quality
- [ ] No dead code or debug artefacts
- [ ] No secrets hardcoded
- [ ] Comments only where WHY is non-obvious

### Security
- [ ] No new SQL/command/XSS injection surface
- [ ] Input validated at system boundaries only

### Scope
- [ ] No changes outside the spec's affected files (or deviation explained)
- [ ] No unasked-for refactors or feature additions

### Test evidence
- [ ] Actual test command was run — not just typecheck/lint
- [ ] Raw output pasted — includes pass count and timing
