---
name: actionable-issues
description: >
  Find the most actionable open issues that an agent can realistically resolve.
  Fetches issues from GitHub or Sentry based on user prompt, filters out issues
  with existing PRs, explores the codebase to validate each issue is real, and
  ranks the top 5-10 by actionability. Use when asked to find actionable issues,
  triage issues, find easy fixes, find good first issues, or identify what to
  work on next.
license: Apache-2.0
compatibility: Requires git, gh CLI. Optional: sentry CLI for Sentry issues.
metadata:
  author: MathurAditya724
  version: "1.0"
---

# Actionable Issues

Find, validate, and rank the most actionable open issues for the current project.
Follow every phase in order. The goal is triage — not resolution. Spend roughly
2-3 minutes of investigation per issue, not more.

## Phase 1: Determine Scope

Parse the user's prompt to determine:

1. **Issue source**: GitHub Issues, Sentry issues, or both.
   If the user doesn't specify, default to GitHub Issues for the current repo.
2. **Filters**: any labels (`bug`, `good first issue`), severity, date range,
   keywords, or component the user mentioned.
3. **Count**: how many to return. Default to the top 5-10 most actionable.

## Phase 2: Fetch Candidate Issues

### GitHub Issues

Try MCP tools first (e.g. `list_issues`). If unavailable, use the `gh` CLI:

```bash
# Fetch open issues with metadata
gh issue list --state open --json number,title,body,labels,assignees,createdAt,comments,url --limit 50

# Fetch open PRs to cross-reference
gh pr list --state open --json number,title,body,headRefName --limit 50
```

Filter out issues that are **not actionable candidates**:

- **Has a linked PR**: check if any open PR references the issue number in its
  title, branch name (`fix/123-...`, `issue-123-...`), or body (`Fixes #123`,
  `Closes #123`, `Resolves #123`). Discard these.
- **Assigned to someone**: skip unless the user explicitly says to include
  assigned issues.
- **Stale**: older than 1 year with no recent comments. Skip unless the user
  asks for them.

### Sentry Issues

Try MCP tools first. If unavailable, use the `sentry` CLI:

```bash
sentry issue list <org/project> --query "is:unresolved" --json --limit 30
```

Filter out:

- Issues already linked to a resolved commit or merged PR (if available)
- Issues where the stack trace points entirely to third-party code
- Issues with zero events in the last 30 days (stale noise)

Prefer issues with higher event counts — they affect more users.

### Combined

If the user asked for both sources, fetch from each independently and merge the
candidates into a single list for triage.

## Phase 3: Initial Triage (Fast Pass)

Quickly scan every candidate. For each issue, answer three questions:

1. **Is it clear?** Can you understand what the problem is from the title and body?
2. **Is it scoped?** Does it look like a contained change (1-3 files), or a
   large refactor / feature requiring product decisions?
3. **Is it reproducible?** Does it include steps, stack traces, or error messages?

### Discard issues that are:

- **Feature requests requiring product decisions** — "should we add dark mode?"
  is not actionable without stakeholder input
- **Discussion threads** — long threads with no clear conclusion or action item
- **Duplicates** — multiple issues describing the same problem; keep the most
  detailed one
- **Infrastructure / access dependent** — requires access to production servers,
  databases, third-party dashboards, or credentials you don't have
- **Too vague to act on** — "it's broken" with no reproduction steps, no error
  messages, and no component identified

After the fast pass, narrow the list to ~15-20 candidates.

## Phase 4: Deep Validation (Thorough Pass)

For each remaining candidate, investigate the codebase.

### Step 1: Locate Relevant Code

Find the files, functions, and code paths mentioned in or implied by the issue.
Use grep, file search, and code reading. For Sentry issues, the stack trace
points directly to the code.

### Step 2: Verify the Issue Is Real

This is the critical step. Issues can be stale, already fixed, or based on
misunderstanding. The level of verification depends on the issue:

**For clear bugs** (stack trace, obvious code defect):
- Trace the code path and confirm the defect exists in the current code
- Check that the described behavior matches what the code does
- No need to write tests at this stage

**For ambiguous issues** (unclear if it's a real bug):
- Write a minimal reproduction test or assertion that would fail if the issue
  is real, and pass if it isn't
- This is a throwaway test for validation only — do not commit it
- Delete the test file after validation

**For issues referencing external behavior** (library bugs, API changes,
browser-specific behavior):
- Search the web to confirm the external behavior described in the issue
- Check if the library version in use is affected
- Check the library's changelog / issue tracker for related fixes

### Step 3: Check if Already Fixed

The issue may be stale:

```bash
# Check recent commits touching the relevant files
git log --oneline -10 -- <relevant-files>
```

If the code has changed since the issue was filed and the described problem no
longer exists, mark the issue as `already-fixed`.

### Step 4: Estimate Complexity

Assess how much work the fix would take:

- How many files need to change?
- Are there existing test patterns to follow, or would you need to set up
  testing infrastructure?
- Is the fix mechanical (e.g. add a null check, fix a typo) or does it require
  design decisions?
- Are there database migrations, API changes, or breaking changes involved?

### Assign a Validation Result

Mark each issue with one of:

| Result | Meaning |
|--------|---------|
| `confirmed` | Defect verified in current code. Clearly fixable. |
| `likely-valid` | Evidence supports the issue but couldn't fully reproduce. |
| `uncertain` | Not enough information to confirm or deny. |
| `likely-false` | Evidence suggests the issue doesn't exist or was misunderstood. |
| `already-fixed` | The code has changed and the issue no longer applies. |

Drop `likely-false` and `already-fixed` issues from the ranking. Mention them
in the summary so the user can close them.

## Phase 5: Rank by Actionability

Score each validated issue. Actionability means "how easily can an agent
resolve this correctly" — not importance to the business.

### Scoring Dimensions

| Factor | Weight | What it measures |
|--------|--------|-----------------|
| **Confidence** | High | Is the issue confirmed real? `confirmed` > `likely-valid` > `uncertain` |
| **Containment** | High | Is the fix localized to 1-3 files, or scattered across the codebase? |
| **Clarity** | Medium | Is the expected behavior unambiguous? No product decisions needed? |
| **Testability** | Medium | Can the fix be verified with tests? Are test patterns already established? |
| **Impact** | Medium | Event count (Sentry), thumbs-up/comments (GitHub), severity labels |
| **Risk** | Low (inverse) | Does the fix touch critical paths, DB schemas, public APIs? Lower risk = more actionable |

Sort by overall actionability. Break ties by impact.

## Phase 6: Output Report

Present the results in this format:

```
## Actionable Issues

### 1. [Issue Title] (#number or Sentry ID)
- **Source**: GitHub Issue / Sentry
- **URL**: <direct link>
- **Validation**: confirmed / likely-valid / uncertain
- **Complexity**: Low / Medium (est. N files to change)
- **Summary**: one-line description of the actual problem
- **Why actionable**: 1-2 sentences — why this is easy for an agent to fix
  (e.g. contained to one file, clear reproduction, existing test patterns)
- **Approach sketch**: 1-3 sentences on how to fix it
- **Risk**: what could go wrong or what to watch out for

### 2. ...
(repeat for top 5-10)
```

### Summary Footer

End the report with:

```
## Triage Summary

- **Scanned**: N issues
- **Filtered out**: M issues (X had existing PRs, Y were stale, Z were unclear)
- **Validated**: P issues
- **Likely false / already fixed**: Q issues (list them so user can close)
- **Ranked**: top K presented above
```

## Gotchas

- **Rate limits**: `gh api` has rate limits. Use `--limit` to cap fetches.
  Don't paginate through hundreds of issues.
- **False positives in PR detection**: an issue comment might mention `#123`
  without being a fix. Check PR titles and branch names, not just body text.
  Also check the PR's linked issues field when available.
- **Stale issues**: an issue open for 2 years may have been silently fixed.
  Always check `git log` on the relevant files before ranking it.
- **Monorepos**: issues may reference a specific package. Scope your code
  exploration to the relevant package directory, not the entire repo.
- **Sentry noise**: high-frequency Sentry issues aren't always actionable.
  They might be caused by infrastructure, third-party code, or user error.
  Check if the stack trace points to first-party code before investigating.
- **Ambitious issues disguised as bugs**: "the search is slow" might look like
  a bug but require an architectural overhaul. Check the complexity before
  ranking it high.
- **Default branch**: always detect with
  `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`.
  Never assume `main`.
- **Throwaway test cleanup**: if you wrote reproduction tests during Phase 4,
  delete them before finishing. Do not leave test files behind.
- **Don't over-invest**: this is triage. If you can't validate an issue in
  ~2-3 minutes, mark it `uncertain` and move on. The user or the resolve-issue
  skill will do the thorough work later.
- **Web search for context**: some issues reference external APIs, library bugs,
  or browser-specific behavior. A quick web search is often faster and more
  reliable than trying to reason about external behavior from code alone.
