# PR Review Automation

Automated first-pass code review for every PR, powered by `anthropics/claude-code-action@v1`.

Every PR gets a verdict-first review (APPROVE / REQUEST_CHANGES / COMMENT) with `file:line` findings across correctness, security, test evidence, and style — so the reviewer approves or rejects instead of reading line-by-line.

## How it works

1. A PR is opened or updated (non-draft)
2. GitHub Actions triggers `.github/workflows/pr-review.yml`
3. Claude reviews the diff in context of the full repo
4. Findings post as PR comments with inline `file:line` citations
5. Verdict line is always first

## Setup for a new repo

1. Add `ANTHROPIC_API_KEY` to repo secrets (Settings → Secrets → Actions)
2. Install [Claude GitHub App](https://github.com/apps/claude) on the repo
3. Copy `.github/workflows/pr-review.yml` into the target repo's `.github/workflows/`
4. Copy `CLAUDE.md` (or write a repo-specific one)
5. Open a test PR — the `PR Review` check run should appear within ~2 minutes

See `REVIEW.md` for the full checklist and tuning guide.

## Tuning

Edit `REVIEW.md` and the `prompt:` block in the workflow to adjust severity thresholds, add repo-specific rules, or change what dimensions are checked.
