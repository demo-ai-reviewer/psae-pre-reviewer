# PSAE pre-reviewer

An AI first reviewer for Odoo PS pull requests, posting as a GitHub account
(the reviewer account, e.g. `odoobot-psae`).

## Flow

1. **Pull request opened, or any push to it (force pushes included):** the
   reviewed repository's small workflow calls `pre-review.yml` here.
2. **Claude Code runs the pre-review** described in
   `.github/actions/pre-reviewer/skill/SKILL.md`:
   - target Odoo version (16.0–20.0) from the branches and manifests;
   - changed XML against the Odoo XML guidelines of that version;
   - final newline of every changed file;
   - the pull request has a description;
   - commit messages: `[TAG][TASK_ID] module: summary` plus a description;
   - every changed module bumps its manifest `version`.
3. **Findings** are posted as inline comments (one per issue), the rest in a
   summary comment. The `Agent Pre-review` check **fails** while any remain.
4. **Every push is reviewed again** until it passes.
5. **On pass**, the workflow (as the reviewer account) requests
   `psae-js-reviewer` when JavaScript changed, otherwise `psae-reviewer`
   (both for a mixture), in the pull request's organization, and checks the
   request went through. It also re-checks the pull request and commit
   descriptions itself, so a miss by the reviewer cannot let them through.

## Files

- `.github/workflows/pre-review.yml` — reusable workflow: checks out the pull
  request, runs the pre-reviewer, re-checks the descriptions, fails unless
  everything passes, then requests the review team(s).
- `.github/actions/pre-reviewer/action.yml` — runs Claude Code with the
  instructions and the allowed tools.
- `.github/actions/pre-reviewer/skill/SKILL.md` — the review instructions.

## Setup in a reviewed repository

Add `.github/workflows/pre-review.yml`:

```yaml
name: PSAE pre-review

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read

jobs:
  pre-review:
    if: ${{ !github.event.pull_request.draft }}
    concurrency:
      group: psae-pre-review-${{ github.event.pull_request.number }}
      cancel-in-progress: true
    uses: demo-ai-reviewer/psae-pre-reviewer/.github/workflows/pre-review.yml@main
    secrets: inherit
```

Secrets (repository or organization): `ODOOBOT_TOKEN`, the reviewer account's
classic token with the `repo` and `read:org` scopes, and
`CLAUDE_CODE_OAUTH_TOKEN`. The reviewer account must be a member of the
organization with write access to the repository; the review teams need
access to the repository. This repository must allow access from the
organization's repositories (Settings → Actions → General → Access).
