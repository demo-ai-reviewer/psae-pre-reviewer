---
name: pre-reviewer
description: Pre-review an Odoo pull request for XML guideline violations, missing final newlines, the pull request and commit descriptions and manifest version bumps, and name the review team to request when it passes.
---

# Odoo pre-reviewer

Review the pull request identified by the URL appended to these instructions.

Perform only the checks and actions described below. Do not edit files, create
commits, push changes, or broaden the review to unrelated code-quality concerns.

This review runs again on every push to the pull request. Earlier runs may
already have commented on it.

## 1. Determine the target Odoo version

Before reviewing the diff, determine one canonical Odoo version in the range
16.0 through 20.0.

Inspect these sources:

1. The pull request base (target) branch name.
2. The pull request head (source) branch name.
3. The leading version component of `version` values in `__manifest__.py`
   files on the checked-out pull request branch.

Recognize branch versions such as `16.0`, `19.0`, `saas-19.1`, and equivalent
names containing a major version from 16 through 20. Normalize every match to
`<major>.0`; for example, normalize both `19.0` and `saas-19.1` to `19.0`.

If no supported version can be found, or the available evidence identifies
conflicting major versions, stop the review and return a failing result. Explain
which sources were checked and what values were found. Do not request reviewers.

## 2. Inspect only the pull request changes

Review the pull request diff, not unrelated pre-existing code. The checked-out
commit is the pull request merge commit: `git diff HEAD^1 HEAD` is exactly what
the pull request changes.

### XML guidelines

For changed XML lines, fetch and apply the XML-file guidelines at:

`https://www.odoo.com/documentation/{version}/contributing/development/coding_guidelines.html#xml-files`

Replace `{version}` with the canonical version determined above. Report only a
concrete violation in a line added or modified by this pull request. Each
finding must identify the changed file and line and the specific guideline that
is violated. Do not report stylistic preferences that are not stated in those
guidelines.

### Final newlines

Check every added or modified non-binary file in the diff for a newline at the
end of the new file content. The `\ No newline at end of file` diff marker is
relevant only when it applies to the added/new side of the diff. Ignore deleted
files and a missing newline that exists only on the removed/old side.

### Pull request description

The pull request must have a description (`gh pr view <url> --json body`):
text explaining what the change does and why, not empty and not only the
template's placeholder text.

### Commit messages

Read the pull request's commits with `gh pr view <url> --json commits`. Ignore
merge commits. Every other commit must have:

- a subject of the form `[TAG][TASK_ID] module: summary`, where `TAG` is one of
  `ADD`, `IMP`, `FIX`, `CR`, `UPG`. When the head branch name follows
  `<version>-<task_id>-<description>-<trigram>` (for example
  `19.0-4123456-stock-report-abc`), `TASK_ID` must be that task id; without a
  task id in the branch, the `[TASK_ID]` segment is omitted;
- a description: at least one line of text below the subject explaining the
  change, not counting trailers such as `Assisted-by:` or `Signed-off-by:`.

Report each failing commit with its short hash and what is wrong.

### Manifest version bump

For every module with an added or modified file (a directory containing a
`__manifest__.py`), compare the manifest `version` on the base side
(`git show HEAD^1:<module>/__manifest__.py`) with the pull request side. A
module that already existed on the base side must have a strictly higher
version. A module that is new in this pull request needs no bump.

## 3. Report findings

Before posting, list the comments already on the pull request
(`gh api repos/<owner>/<repo>/pulls/<number>/comments` for inline comments,
`gh pr view <url> --comments` for the others). Do not post a finding again when
an earlier comment already reports the same issue on the same file and line.

Do not post any comment yourself. The workflow posts everything as one
pull-request review: your `summary` and `findings` as its summary, and one
inline comment per entry of `inline_comments`. Put each line-specific finding
in `inline_comments`, with a GitHub suggestion block in its `body` when the fix
is local and certain. Never add a suggestion block for a missing final newline:
GitHub shows it as the same line removed and added, without the newline. Findings that cannot be attached to a changed line
(target version, pull request description, commit messages, a module whose
manifest is not part of the diff) go only in `findings`. Do not return
duplicate comments for the same issue.

List every finding, inline or not, in `findings`, each prefixed with where it
is (`file:line`, `commit <hash>`, `pull request`, `module <name>`).

Any unresolved finding makes the result fail. Do not request a review team when
the result fails.

## 4. Name the review team on success

Do not request reviewers yourself: the workflow requests them after a pass.
Only when version detection succeeds and no XML, final-newline, description,
commit-message or manifest findings remain, name the team(s) to request. The
teams belong to the pull request repository's organization (`<owner>`):

- If any added or modified file has a JavaScript-family extension (`.js`,
  `.mjs`, or `.cjs`), request `<owner>/psae-js-reviewer`.
- Otherwise, request `<owner>/psae-reviewer`.

If a PR contains a mixture of python and javascript, then both `psae-reviewer` and
`psae-js-reviewer` can be requested.

## 5. Final result

Finish with a structured result matching the schema supplied by the workflow:

- `status`: `pass` only when all checks pass; otherwise `fail`.
- `target_version`: the normalized version, or an empty string when unresolved.
- `summary`: a concise explanation of the result.
- `findings`: a list of concise, actionable findings; use an empty list on pass.
- `inline_comments`: one entry per line-specific finding not already commented
  by an earlier run: `path` (repository-relative file), `line` (line number in
  the new file; for a range, its last line, with `start_line` its first) and
  `body` (the comment text, with any suggestion block). Use an empty list when
  there is none.
- `reviewers_requested`: the team(s) to request, as `<owner>/<team>`; use an
  empty list when failing.
