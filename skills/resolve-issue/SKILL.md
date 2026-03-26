---
name: resolve-issue
description: >
  Resolve GitHub or Sentry issues end-to-end: fetch issue details, create a
  branch, plan changes, implement the fix following codebase conventions,
  self-review, commit with human-style messages, and open a draft PR.
  Use when given a GitHub issue URL, Sentry issue URL, or asked to fix/resolve
  an issue.
license: MIT
compatibility: "Requires git, gh CLI. Optional: sentry CLI for Sentry issues."
metadata:
  author: MathurAditya724
  version: "1.0"
---

# Resolve Issue

End-to-end workflow for taking an issue from URL to draft PR. Follow every phase
in order. Do not skip phases or combine them.

## Phase 1: Issue Intelligence Gathering

Determine the issue source from the URL or user prompt, then fetch full details.

### GitHub Issues

Try MCP tools first (e.g. `get_issue`). If MCP is unavailable, fall back to the
`gh` CLI:

```bash
gh issue view <url-or-number> --json number,title,body,labels,assignees,comments,state
```

### Sentry Issues

Try MCP tools first. If unavailable, fall back to the `sentry` CLI:

```bash
sentry issue view <PROJECT-ID>
sentry issue plan <PROJECT-ID>
```

Use the short ID format (`PROJECT-123`), not the numeric ID.

### Early Exit Checks

Before proceeding, verify the issue is actually open and not already addressed:

- If the issue is **closed or resolved**, stop and inform the user.
- Check for **existing open PRs** that already reference this issue:

```bash
gh pr list --state open --search "<issue-number>" --json number,title,headRefName,url
```

If a PR already addresses the issue, stop and share the PR link with the user
instead of creating a duplicate.

### What to Extract

From the issue, identify and record:

- **Problem statement** — what is broken or missing
- **Reproduction steps** — how to trigger the issue
- **Error output** — stack traces, error messages, screenshots
- **Expected behavior** — what should happen instead
- **Labels/tags** — bug, feature, severity, component
- **Linked PRs or issues** — related context
- **Discussion** — relevant comments that clarify scope or constraints

If the issue is vague, search the web for related error messages or library
behaviors before proceeding.

Summarize the issue in 2-3 sentences before moving on. This summary anchors
every subsequent phase.

## Phase 2: Codebase Context Discovery

Before touching any code, learn the repo's conventions.

### Contribution Guidelines

Check these paths in order. Use the first one found:

1. `CONTRIBUTING.md`
2. `.github/CONTRIBUTING.md`
3. `docs/CONTRIBUTING.md`

Extract: branch naming, commit message format, PR process, testing requirements.

### PR Template

Check these paths:

1. `.github/PULL_REQUEST_TEMPLATE.md`
2. `.github/pull_request_template.md`
3. `.github/PULL_REQUEST_TEMPLATE/` (directory of templates)

If a template exists, you will use it verbatim in Phase 8.

### Commit Convention

Look for commit style in this order:

1. Config files: `.commitlintrc`, `.commitlintrc.json`, `.commitlintrc.yaml`,
   `.commitlintrc.yml`, `commitlint.config.js`, `commitlint.config.ts`
2. `package.json` — check for `commitlint` config or `standard-version`/
   `semantic-release` in dependencies
3. `CONTRIBUTING.md` — may document the convention
4. Recent history: `git log --oneline -20` — infer from existing messages

Common conventions to detect:

- **Conventional Commits**: `type(scope): description` (e.g. `fix(auth): ...`)
- **Angular**: same as conventional commits
- **Gitmoji**: emoji prefix (`:bug:`, `:sparkles:`)
- **Freeform**: just match the tone and length of existing messages

### Branch Naming

Check `CONTRIBUTING.md` first. If not documented, infer from existing branches:

```bash
git branch -r --sort=-committerdate | head -20
```

Default pattern if nothing is documented:

- Bug fix: `fix/<issue-number>-<short-kebab-description>`
- Feature: `feat/<issue-number>-<short-kebab-description>`

### Project Structure

Understand the codebase layout: framework, language, directory structure,
testing patterns. Read the entry point files and the test directory to learn
the conventions before writing code.

## Phase 3: Branch Creation

```bash
# Always detect the default branch — do not assume "main"
DEFAULT_BRANCH=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')

# Ensure you're up to date
git fetch origin
git checkout "$DEFAULT_BRANCH"
git pull origin "$DEFAULT_BRANCH"

# Create the branch
git checkout -b <branch-name>
```

Use the naming convention discovered in Phase 2. Include the issue number.

If a branch with that name already exists, append a suffix (`-v2`, `-alt`) rather
than force-overwriting.

## Phase 4: Planning

Before writing any code, create a plan using a todo list:

1. List every file that needs to change
2. List every test file that needs to be added or modified
3. Note edge cases raised in the issue
4. Note any dependencies or migrations needed
5. Identify the order of changes (e.g. schema first, then logic, then tests)

Keep the plan visible so you can track progress.

## Phase 5: Implementation

Make changes following the codebase's existing patterns. Match the style of
neighboring code, not your own preferences.

### Style Matching

- **Indentation**: tabs vs spaces, width — match the file you're editing
- **Naming**: camelCase, snake_case, PascalCase — match the codebase
- **Imports**: ordering, grouping, absolute vs relative — match existing files
- **Error handling**: match the patterns used in similar functions
- **File organization**: if the repo puts tests next to source files, do the same.
  If tests are in a separate directory, follow that convention.

### Testing

- Add tests that cover the fix. Follow the existing test framework and patterns.
- If the issue includes a reproduction case, turn it into a test.
- Run the existing test suite to confirm nothing is broken:

```bash
# Detect and run the project's test command
# Check package.json scripts, Makefile, or CI config for the right command
```

### Linting and Formatting

If the project has a linter/formatter configured, run it on changed files:

```bash
# Check package.json scripts, Makefile, or CI for lint/format commands
```

Fix any linting errors before proceeding.

## Phase 6: Self-Review

Review your own diff before committing:

```bash
git diff
```

Check for:

- [ ] **Completeness** — does the change fully resolve the issue?
- [ ] **No unrelated changes** — no stray whitespace fixes, no refactors
- [ ] **No debug artifacts** — no `console.log`, `print()`, `debugger`, `TODO`
- [ ] **Error handling** — are failure cases covered?
- [ ] **Test coverage** — is the fix tested? Are edge cases from the issue covered?
- [ ] **No secrets** — no hardcoded tokens, keys, passwords, or PII
- [ ] **No leftover comments** — no "AI generated" or "Claude" markers

If anything fails the review, fix it now. Do not proceed with known issues.

## Phase 7: Commit

### Staging

Stage only the files related to the fix:

```bash
git add <specific-files>
```

Never use `git add .` blindly. Review what you're staging.

### Commit Message

Write the message following the convention detected in Phase 2. The message must:

- Reference the issue (e.g. `fixes #123` or `Resolves PROJECT-123`)
- Focus on **why** the change was made, not just what changed
- Read like a human wrote it — natural language, no boilerplate
- Be concise: imperative mood, ~50 char subject line, optional body

**Good example (Conventional Commits):**

```
fix(auth): prevent session expiry during active requests

The session TTL was calculated from login time, not last activity.
Users making continuous requests would still get logged out after
the fixed timeout. Now the TTL resets on each authenticated request.

Fixes #342
```

**Bad example:**

```
fix: fixed the bug in auth

- Updated session.ts
- Added test
- Resolves #342
```

If the change is logically distinct (e.g. a migration + the fix + a test), use
separate commits with clear messages for each.

## Phase 8: Draft PR Creation

### Push

```bash
git push -u origin <branch-name>
```

### Create the PR

```bash
gh pr create --draft --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

### PR Title

- Follow the convention from the PR template or CONTRIBUTING.md
- If none exists, use the commit convention for the title
  (e.g. `fix(auth): prevent session expiry during active requests`)
- Keep it concise and descriptive

### PR Body

If a PR template exists, fill it in completely. Preserve HTML comments and
section headers from the template.

If no template exists, use this structure:

```markdown
## Summary

Brief description of what this PR does and why.

## Problem

What was wrong. Link to the issue: `Fixes #<number>` (or Sentry issue URL).

## Changes

- What was changed and why (not a file list, but a logical description)

## Testing

How the fix was tested. Mention specific test cases added.

## Notes for Reviewers

Anything the reviewer should pay attention to, or decisions you made that
might not be obvious.
```

Write in a natural, human tone. Avoid bullet-point-only descriptions. Use
`Fixes #<number>` in the body so the issue auto-closes when merged.

### After PR Creation

Output the PR URL so the user can review it.

## Gotchas

- **Default branch**: never assume `main`. Detect with `gh repo view --json defaultBranchRef`.
- **Existing branches**: check before creating. `git branch -a | grep <name>`.
- **Squash repos**: if the repo uses squash merging (check `.github/settings.yml`
  or repo settings), the PR title becomes the commit message — make it count.
- **PR templates with HTML comments**: templates often use `<!-- -->` for
  instructions. Preserve the structure, replace the placeholder content.
- **Monorepos**: scope your changes to the correct package. Don't modify the
  root unless the issue is at the root level.
- **CI checks**: after pushing, note if CI fails. If it fails on your changes,
  fix them. If it fails on unrelated flaky tests, mention it in the PR body.
- **Sentry issue IDs**: use `PROJECT-123` format (short ID), not the numeric ID.
- **Linked PRs**: if another PR already addresses the issue, stop and inform the
  user instead of creating a duplicate.
- **Inferring conventions from history**: when no CONTRIBUTING.md exists, run
  `git log --oneline -20` and match the style of existing commits.
