# Agentic Development Pipeline v1

**Scope:** Repo-agnostic. Adding a new repo is configuration (register branch protection rules, CI command, test runner) — not invention.

---

## 1. Entry Criteria for an Agent-Ready Issue

An issue is agent-ready when ALL of the following are true:

| Criterion | Check |
|---|---|
| Single bounded outcome | One user-facing behaviour changed or one bug fixed; not "improve X" |
| Acceptance criteria written | ≥1 explicit, falsifiable AC statement on the issue |
| Scope fits one PR | No more than ~400 LOC changed (estimate); larger work is split first |
| No blocking dependencies | All upstream issues are `done` or `in_review` |
| Repo config registered | Repo has a registered CI command, test runner, and branch-protection rule |
| No secrets required | Agent can execute without human-held credentials not in the vault |

If any criterion fails, the issue stays in `backlog` or `blocked` until resolved by the founder or a planning agent.

---

## 2. Spec Template

Every agent-ready issue must include a filled spec block before implementation begins.

```
## Spec

### Context
<1-3 sentences: what problem this solves and why now>

### Acceptance Criteria
- [ ] <falsifiable statement — user can do X / system returns Y / error Z is gone>
- [ ] <…>

### Out of Scope
- <explicit exclusions to prevent scope creep>

### Affected Files / Routes (best guess)
- <file or route pattern — agent will verify>

### Test Evidence Required
- <what command to run and what output proves AC met>
  e.g. `pytest tests/test_auth.py -v` — all tests pass, including `test_login_success`

### Repo Config
- repo: <repo-name>
- ci_command: <e.g. `npm test` / `pytest` / `cargo test`>
- branch_prefix: <e.g. `feat/` / `fix/`>
```

---

## 3. Implementation Rules

1. **Feature branches only.** Every change lands on a branch named `{prefix}/{issue-id}-{slug}` (e.g. `feat/APO-3-pipeline-design`). Direct commits to `main` are forbidden by branch protection.
2. **One issue → one branch → one PR.** Do not bundle unrelated changes.
3. **No force-pushes to `main`.** Rebase or merge only via PR.
4. **CI must pass before merge.** The registered `ci_command` must exit 0 with actual test output attached to the PR.
5. **Self-review before opening PR.** See checklist in §4.
6. **PR opened as draft until self-review complete.** Promote to ready-for-review only after checklist is signed off.

---

## 4. Self-Review Checklist

The implementing agent runs this before promoting the PR from draft:

```
## Self-Review

### Correctness
- [ ] Each AC from the spec is met and evidenced by test output
- [ ] No regression in existing tests (full suite output attached)
- [ ] Edge cases named in spec are handled

### Code Quality
- [ ] No dead code or debug artefacts left in
- [ ] No new secrets or credentials hardcoded
- [ ] Comments added only where WHY is non-obvious

### Security
- [ ] No new SQL/command/XSS injection surface
- [ ] Input validated at system boundaries only (not internal calls)

### Scope
- [ ] No changes outside the affected files listed in spec (or deviation explained)
- [ ] No unasked-for refactors or feature additions

### Test Evidence
- [ ] Actual test command was run (not just typecheck/lint)
- [ ] Raw output pasted — includes pass count and timing
- [ ] Output matches expected AC outcomes
```

---

## 5. Test Evidence Requirements

**"Tested" means:** a test runner (pytest, jest, cargo test, go test, vitest, rspec, etc.) was actually executed and its stdout/stderr is attached.

**"Tested" does NOT mean:** `tsc --noEmit`, `eslint`, `cargo check`, `mypy`, or any other compile/lint/format tool.

### Evidence format

Paste the raw terminal block, not a summary:

```
$ pytest tests/ -v
========================= test session starts ==========================
collected 12 items

tests/test_auth.py::test_login_success PASSED                   [  8%]
tests/test_auth.py::test_login_wrong_password PASSED            [ 16%]
...
========================= 12 passed in 1.43s ===========================
```

If the test suite doesn't cover the new behaviour, a new test must be added before the PR is ready. No AC can be marked done without corresponding passing test output.

---

## 6. PR Description Standard

```markdown
## Summary
<1-3 bullets: what changed and why>

## Acceptance Criteria
- [x] <AC 1 — copy from spec, mark done>
- [x] <AC 2>

## Test Evidence
\`\`\`
<paste raw test runner output>
\`\`\`

## Self-Review
[x] Self-review checklist complete (see checklist comment or inline)

## Out of Scope (confirmed)
- <restate what was explicitly excluded>

## Remaining Risk / Follow-up
- <anything the reviewer should know that isn't covered by tests>
```

---

## 7. Re-Review Loop Handling

| Reviewer action | Agent response |
|---|---|
| `approved` with no comments | Merge immediately (squash). Mark issue `done`. |
| `approved` with nits | Fix nits, push to same branch, re-request review. No new PR. |
| `changes_requested` — clear fix | Fix, push, add a comment summarising what changed and why, re-request review. |
| `changes_requested` — scope question | Do NOT implement. Comment asking for clarification, set issue `blocked`, name the reviewer as unblock owner. |
| Second `changes_requested` on same point | Escalate to founder: post the conflict as an issue comment, set `blocked`, tag founder. |
| Reviewer silent > 48 h | Ping reviewer in issue comment; if no response after 24 h more, escalate to founder. |

**Loop cap:** If a PR has ≥3 re-review cycles, auto-escalate to founder with a summary of the disagreement. Do not spin indefinitely.

---

## 8. Metrics

| Metric | Definition | Source |
|---|---|---|
| **Defect-escape rate** | Bugs filed against issues already marked `done`, per sprint | Issue tracker: bugs with `regression` label |
| **Review turnaround** | Time from PR `ready-for-review` to first reviewer action (h) | PR timestamps |
| **Rework loops** | Number of `changes_requested` cycles per PR | PR review history |
| **Time-to-merge** | Time from branch creation to merge (h) | Git + PR timestamps |
| **Agent self-review hit rate** | % of PRs where self-review checklist was complete before human review | PR description audit |
| **AC coverage** | % of merged PRs with passing test evidence attached | PR audit |

Review metrics weekly. If rework loops > 1.5 average, audit the spec template and entry criteria.

---

## 9. Repo Configuration Schema

```yaml
# .paperclip/repo-config.yaml  (or equivalent registry)
repo: <repo-name>
ci_command: <shell command that runs the full test suite>
test_runner: <pytest | jest | cargo test | go test | …>
branch_protection:
  main: true           # no direct pushes
  require_pr: true
  require_ci_pass: true
branch_prefix_map:
  feature: feat/
  bug: fix/
  chore: chore/
  docs: docs/
```

Adding a new repo = add one entry to this registry. No pipeline logic changes.
