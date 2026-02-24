# Skills

Personal Claude Code skills plugin marketplace.

## Setup

Add this marketplace and install plugins:

```
/plugin marketplace add brhannan/skills
/plugin install expert-review@bhannan-skills
```

## Available Plugins

### expert-review

Multi-agent code review deploying 12 specialized agents across 6 domains with consensus-based confidence scoring. Inspired by [Anthropic's multi-agent research architecture](https://www.anthropic.com/engineering/multi-agent-research-system).

**Usage:**

```
/expert-review https://github.com/owner/repo/pull/123
/expert-review ./src/auth/ --profile=minimal
/expert-review --staged
/expert-review --json
```

**Arguments:**

| Argument | Default | Description |
|----------|---------|-------------|
| `<target>` | uncommitted changes | GitHub PR URL, file/directory path, or omit for local changes |
| `--profile` | `full` | `full` (12 agents), `lite` (6), or `minimal` (3) |
| `--staged` | `false` | Review only staged changes |
| `--reviewer-model` | `sonnet` | Model for reviewers: `haiku`, `sonnet`, `opus` |
| `--post_to_github` | `false` | Post report as PR comment |
| `--json` | `false` | Output machine-readable JSON |
