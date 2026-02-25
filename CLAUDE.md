# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin marketplace (`pipecat-skills`) containing skills for building and deploying Pipecat applications. Published at `pipecat-ai/skills` on GitHub.

## Architecture

This repo follows the Claude Code plugin marketplace pattern (same as `anthropics/skills`):

- `.claude-plugin/marketplace.json` — Marketplace manifest. Defines plugins that each select skills from the shared `skills/` directory.
- `skills/<name>/SKILL.md` — Each skill is a directory with a `SKILL.md` file containing frontmatter (`name`, `description`) and instructions that Claude follows when the skill is invoked.

### Plugins

The marketplace exposes three plugins:

- **`pipecat`** — General Pipecat development skills (e.g. `init`). Invoked as `/pipecat:<skill>`.
- **`pipecat-cloud`** — Pipecat Cloud deployment skills (e.g. `deploy`). Invoked as `/pipecat-cloud:<skill>`.
- **`pipecat-mcp-server`** — Pipecat MCP server skills (e.g. `talk`). Invoked as `/pipecat-mcp-server:<skill>`.

When adding a new skill, create `skills/<name>/SKILL.md` and add it to the appropriate plugin's `skills` array in `marketplace.json`.

## Writing Skills

Skills are markdown files with instructions for Claude. Key conventions:

- Use AskUserQuestion for choices with a small fixed set of options (2-4 choices like yes/no, bot type, pipeline mode).
- For choices with many options (services, transports), show the full list as formatted text and let the user reply in chat.
- Skills that wrap CLI tools should use non-interactive/automation flags rather than interactive prompts, since Claude can't interact with TTY prompts.
- Use `--list-options` style discovery commands when available to avoid hardcoding values that change over time.

## Installation (for end users)

```
/plugin marketplace add pipecat-ai/skills
/plugin install pipecat@pipecat-skills
/plugin install pipecat-cloud@pipecat-skills
```
