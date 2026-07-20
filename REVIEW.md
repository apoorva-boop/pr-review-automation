# PR Review Checklist

Living document. Update this file as patterns improve — it feeds directly into the automated review prompt.

---

## Verdict-first format

Every automated review opens with a single verdict line:

```
## Verdict
APPROVE | REQUEST_CHANGES | COMMENT
```

Then a 1-3 bullet summary, then file:line findings. This lets the founder approve/reject at a glance without reading line-by-line.

---

## What the automated review checks

| Dimension | What it looks for |
|---|---|
| **Correctness** | Logic errors, off-by-one, null/undefined/empty handling, wrong assumptions |
| **Security** | Injection vectors (SQL, command, XSS), hardcoded secrets, improper auth/authz, OWASP Top 10 |
| **Test evidence** | Are ACs covered by tests? Is raw test-runner output attached to the PR? |
| **Style** | Does the code follow `CLAUDE.md` conventions? |
| **Scope** | Changes outside the spec, unasked-for refactors, dead code left in |

---

## Severity levels

| Marker | Meaning | Merge action |
|---|---|---|
| 🔴 blocking | Must fix before merge | Fix, push, re-request review |
| 🟡 nit | Minor, worth fixing | Author's discretion |
| 🔵 suggestion | Improvement idea, uncertain finding | No action required |

---

## Human review still required for

The automated pass is a first filter. A human must still review:

- **Architectural decisions** — does this approach scale / fit the system?
- **Business correctness** — does it solve the right problem?
- **Second `changes_requested` cycle** — if the same point comes back a second time, escalate to founder

Cap: if a PR accumulates ≥3 re-review cycles, escalate with a summary of the disagreement. Do not spin.

---

## Tuning

- Change **what gets flagged**: edit the `prompt:` block in `.github/workflows/pr-review.yml`
- Change **severity thresholds**: add a `REVIEW.md` section at repo root following the Claude Code Review spec
- Change **model or turn limit**: edit `claude_args:` in the workflow
- Add **repo-specific rules**: append to this file under a "## Repo-specific checks" heading

---

## Setup checklist (per repo)

- [ ] Add `ANTHROPIC_API_KEY` to repo secrets (Settings → Secrets → Actions)
- [ ] Install [Claude GitHub App](https://github.com/apps/claude) on the repo
- [ ] Copy `.github/workflows/pr-review.yml` into the target repo
- [ ] Copy `CLAUDE.md` or write a repo-specific one
- [ ] Open a test PR and verify the `PR Review` check run appears
- [ ] Confirm findings post as inline comments and the verdict line is first
