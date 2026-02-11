# Pipecat Skills

This repo is a Claude Code plugin and marketplace containing skills for building and deploying Pipecat applications.

## Structure

- `.claude-plugin/plugin.json` - Plugin manifest
- `.claude-plugin/marketplace.json` - Marketplace manifest
- `skills/` - Skill definitions, each in its own subdirectory with a `SKILL.md`

## Skills

- `skills/deploy/` - Deploy a Pipecat application to Pipecat Cloud

## Installation

Developers can install these skills in their projects:

```
/plugin marketplace add pipecat-ai/skills
/plugin install pipecat-cloud@pipecat-skills
```

The deploy skill will then be available as `/pipecat-cloud:deploy`.
