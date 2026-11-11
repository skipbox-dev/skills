# Skipbox Skills

A skill that teaches AI coding agents how to move GitHub Actions jobs to [Skipbox](https://skipbox.dev): cheap, fast GitHub Actions runners, unlimited for a flat monthly price.

Ask your agent to "make our GitHub Actions cheaper" or "move CI to Skipbox", and it finds the jobs worth moving, changes their `runs-on` line, adds the setup steps the job needs, and opens a pull request. You install the GitHub App; the agent does the rest.

## Install

### skills.sh (Claude Code, Codex, Cursor and other agents)

```bash
npx skills add skipbox-dev/skills
```

### Claude Code plugin

```
/plugin marketplace add skipbox-dev/skills
/plugin install skipbox@skipbox-skills
```

### By hand

Copy [`skills/skipbox-github-runners/SKILL.md`](skills/skipbox-github-runners/SKILL.md) into your agent's skills or rules folder (for Claude Code: `.claude/skills/skipbox-github-runners/SKILL.md`).

## What's in it

| Skill | What it covers |
|---|---|
| [`skipbox-github-runners`](skills/skipbox-github-runners/SKILL.md) | Which jobs to move, the one-line `runs-on` change, labels and sizes, what the runner image has, the caches that stay between jobs, how many jobs run at once, and fixes for a job that fails or stays queued |

The skill states no prices. It tells the agent to read them from [skipbox.dev/pricing.md](https://skipbox.dev/pricing.md), so they are current.

## Links

- Docs: https://docs.skipbox.dev (every page as Markdown: add `.md`; index at [`llms.txt`](https://docs.skipbox.dev/llms.txt))
- Pricing: https://skipbox.dev/pricing
- Support: hello@skipbox.dev

Skipbox is run by Fluxenic Inc.
