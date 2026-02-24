---
name: expert-review
description: Multi-agent code review deploying 12 specialized agents across 6 domains with consensus-based confidence scoring. Supports GitHub PRs, local files, and uncommitted changes.
allowed-tools: Task, Read, Write, Bash, Glob, Grep
user-invocable: true
argument-hint: "[<target>] [--profile=full|lite|minimal] [--staged] [--post_to_github] [--json]"
---

# Expert Review

A multi-agent code review system inspired by [Anthropic's multi-agent research architecture](https://www.anthropic.com/engineering/multi-agent-research-system). A coordinator orchestrates 12 specialized reviewer subagents across 6 domains, synthesizes their findings via 5 domain synthesizers, and produces a consensus-based final report with confidence scoring.

## Arguments

- `<target>` (optional): What to review. Can be:
  - A GitHub PR URL (e.g., `https://github.com/owner/repo/pull/123`)
  - A local file or directory path (e.g., `./src/auth/`)
  - Omitted: reviews uncommitted changes (or staged changes with `--staged`)
- `--profile=full|lite|minimal` (default: `full`): Controls how many agents are deployed
  - `full`: 12 agents across all 6 domains (highest quality, ~15x tokens)
  - `lite`: 6 agents, 1 per domain (balanced coverage)
  - `minimal`: 3 agents — bug-hunter, readability, security (fast, cheap)
- `--staged` (default: false): Review only staged changes (`git diff --cached`). Useful for self-review before committing.
- `--reviewer-model=<model>` (default: `sonnet`): Model for reviewer subagents. Options: `haiku`, `sonnet`, `opus`.
- `--post_to_github` (default: false): Post the final report as a comment on the GitHub PR.
- `--json` (default: false): Output machine-readable JSON to stdout (see `references/json-schema.md`).

## Target Resolution (priority order)

1. **GitHub PR URL** → fetch diff via `gh pr diff <number> -R <owner/repo>`
2. **Explicit file/directory path** → review those files directly
3. **`--staged` flag** (no target) → `git diff --cached`
4. **No target, no flag** → `git diff` (all uncommitted changes)

## Instructions

You are the entry point for the expert-review skill. Your job is to:

1. **Parse arguments** from `$ARGUMENTS`:
   - Extract `<target>`, `--profile`, `--staged`, `--reviewer-model`, `--post_to_github`, `--json`
   - Apply defaults for any missing arguments

2. **Resolve the target** using the priority order above:
   - For PR URLs: extract owner, repo, and PR number from the URL
   - For file paths: verify the path exists
   - For staged/uncommitted: verify you're in a git repository

3. **Delegate to the coordinator** by reading `agents/coordinator.md` and launching it as a Task:
   ```
   Task(subagent_type="general-purpose", prompt=<coordinator instructions + resolved parameters>)
   ```

4. **Pass these parameters to the coordinator**:
   - `target_type`: one of `pr`, `file`, `staged`, `uncommitted`
   - `target_value`: the PR URL, file path, or empty string
   - `pr_number`: extracted PR number (if applicable, otherwise "local")
   - `pr_repo`: owner/repo (if applicable)
   - `profile`: full, lite, or minimal
   - `reviewer_model`: haiku, sonnet, or opus
   - `post_to_github`: true/false
   - `json_output`: true/false
   - `skill_dir`: absolute path to this skill's directory (for reading agent .md files)

5. **Display the final report** to the user when the coordinator completes.

## Example Invocations

```
/expert-review https://github.com/myorg/myapp/pull/42
/expert-review ./src/auth/ --profile=minimal
/expert-review --staged --profile=lite
/expert-review --json
/expert-review https://github.com/myorg/myapp/pull/42 --post_to_github --profile=full
```
