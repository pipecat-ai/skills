# Pipecat Skills

This repo is a Claude Code plugin and marketplace containing skills for building and deploying Pipecat applications.

## Structure

- `.claude-plugin/marketplace.json` - Marketplace manifest
- `skills/` - Skill definitions, each in its own subdirectory with a `SKILL.md`

## Skills

- `skills/deploy/` - Deploy a Pipecat application to Pipecat Cloud
- `skills/init/` - Scaffold a new Pipecat project with guided setup

## Installation

Developers can install these skills in their projects:

```
/plugin marketplace add pipecat-ai/skills
/plugin install pipecat@pipecat-skills
/plugin install pipecat-cloud@pipecat-skills
```
