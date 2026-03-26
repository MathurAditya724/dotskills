# dotskills

A shareable collection of [Agent Skills](https://agentskills.io) for issue triage and resolution workflows.

These skills give AI coding agents a structured, repeatable process for finding actionable issues and resolving them end-to-end — from issue URL to draft PR.

## Skills

| Skill | Description | Pattern |
|-------|-------------|---------|
| [resolve-issue](skills/resolve-issue/) | Takes a GitHub or Sentry issue URL and resolves it end-to-end: fetches details, creates a branch, plans and implements the fix following codebase conventions, self-reviews, commits with human-style messages, and opens a draft PR. | Instructions only |
| [actionable-issues](skills/actionable-issues/) | Finds the most actionable open issues an agent can realistically resolve. Fetches issues from GitHub or Sentry, filters out ones with existing PRs, validates each issue against the actual codebase, and ranks the top 5-10 by actionability. | Instructions only |

## Compatibility

These skills follow the open [Agent Skills specification](https://agentskills.io/specification) and work with any compatible agent:

[OpenCode](https://opencode.ai/) ·
[Claude Code](https://claude.ai/code) ·
[VS Code Copilot](https://code.visualstudio.com/) ·
[Cursor](https://cursor.com/) ·
[Gemini CLI](https://geminicli.com/) ·
[OpenAI Codex](https://developers.openai.com/codex) ·
[Amp](https://ampcode.com/) ·
[Goose](https://block.github.io/goose/) ·
[Roo Code](https://roocode.com/) ·
[and many more](https://agentskills.io/home)

## Prerequisites

- **git** and **[gh CLI](https://cli.github.com/)** — required for both skills
- **[sentry CLI](https://cli.sentry.dev/)** — optional, only needed for Sentry issue workflows

## Installation

### Option 1: Cross-client (recommended)

Symlink into `~/.agents/skills/` so any compatible agent can discover them:

```bash
# Create the directory if it doesn't exist
mkdir -p ~/.agents/skills

# Symlink each skill
ln -s /path/to/dotskills/skills/resolve-issue ~/.agents/skills/resolve-issue
ln -s /path/to/dotskills/skills/actionable-issues ~/.agents/skills/actionable-issues
```

### Option 2: Agent-specific

Symlink into your agent's skill directory:

```bash
# OpenCode
mkdir -p ~/.config/opencode/skills
ln -s /path/to/dotskills/skills/resolve-issue ~/.config/opencode/skills/resolve-issue
ln -s /path/to/dotskills/skills/actionable-issues ~/.config/opencode/skills/actionable-issues

# Claude Code
mkdir -p ~/.claude/skills
ln -s /path/to/dotskills/skills/resolve-issue ~/.claude/skills/resolve-issue
ln -s /path/to/dotskills/skills/actionable-issues ~/.claude/skills/actionable-issues
```

### Option 3: Project-level

Copy or symlink into a specific project's `.agents/skills/` directory:

```bash
# From your project root
mkdir -p .agents/skills
ln -s /path/to/dotskills/skills/resolve-issue .agents/skills/resolve-issue
ln -s /path/to/dotskills/skills/actionable-issues .agents/skills/actionable-issues
```

## Usage

### Resolve an issue

Give the agent a GitHub or Sentry issue URL:

```
Resolve this issue: https://github.com/org/repo/issues/123
```

```
Fix this Sentry issue: PROJECT-456
```

The agent will follow the full workflow: fetch the issue, understand the codebase conventions, create a branch, implement the fix, self-review, commit, and open a draft PR.

### Find actionable issues

Ask the agent to find issues worth working on:

```
Find the most actionable issues in this repo
```

```
What are the easiest open bugs to fix?
```

```
Find actionable Sentry issues for my-org/my-project
```

The agent will scan open issues, validate them against the codebase, and return a ranked report of the top 5-10 most actionable ones.

## Creating Your Own Skills

Each skill is a directory with a `SKILL.md` file:

```
my-skill/
├── SKILL.md          # Required: YAML frontmatter + markdown instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: additional documentation
└── assets/           # Optional: templates, data files
```

See the [Agent Skills specification](https://agentskills.io/specification) for the full format and [best practices](https://agentskills.io/skill-creation/best-practices) for writing effective skills.

## License

[Apache-2.0](LICENSE)
