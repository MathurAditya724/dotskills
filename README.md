# dotskills

A shareable collection of [Agent Skills](https://agentskills.io) for issue triage and resolution workflows.

These skills give AI coding agents a structured, repeatable process for finding actionable issues and resolving them end-to-end — from issue URL to draft PR.

## Skills

| Skill | Description |
|-------|-------------|
| [resolve-issue](skills/resolve-issue/) | Takes a GitHub or Sentry issue URL and resolves it end-to-end: fetches details, creates a branch, plans and implements the fix following codebase conventions, self-reviews, commits with human-style messages, and opens a draft PR. |
| [actionable-issues](skills/actionable-issues/) | Finds the most actionable open issues an agent can realistically resolve. Fetches issues from GitHub or Sentry, filters out ones with existing PRs, validates each issue against the actual codebase, and ranks the top 5-10 by actionability. |

## Prerequisites

- **git** and **[gh CLI](https://cli.github.com/)** — required for both skills
- **[sentry CLI](https://cli.sentry.dev/)** — optional, only needed for Sentry issue workflows

## Installation

Install using [`npx skills`](https://github.com/vercel-labs/skills) — the open agent skills CLI that handles installation across 40+ agents automatically.

### Install all skills

```bash
npx skills add MathurAditya724/dotskills
```

You'll be prompted to select which skills and agents to install to.

### Install a specific skill

```bash
npx skills add MathurAditya724/dotskills --skill resolve-issue
npx skills add MathurAditya724/dotskills --skill actionable-issues
```

### Install to specific agents

```bash
# Install to OpenCode and Claude Code
npx skills add MathurAditya724/dotskills -a opencode -a claude-code

# Install to Cursor
npx skills add MathurAditya724/dotskills -a cursor
```

### Install globally (available across all projects)

```bash
npx skills add MathurAditya724/dotskills -g
```

See the full [`npx skills` documentation](https://github.com/vercel-labs/skills) for all options.

## License

[MIT](LICENSE)
