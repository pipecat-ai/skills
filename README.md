# Pipecat Skills for Claude Code

[Skills](https://docs.anthropic.com/en/docs/claude-code/skills) for building and deploying [Pipecat](https://www.pipecat.ai/) applications with Claude Code.

## Installation

Register this repository as a Claude Code Plugin marketplace:

```
/plugin marketplace add pipecat-ai/skills
```

Then install the plugins:

```
/plugin install pipecat@pipecat-skills
/plugin install pipecat-cloud@pipecat-skills
/plugin install pipecat-dev@pipecat-skills
/plugin install pipecat-mcp-server@pipecat-skills
```

Or browse and install interactively:

1. Select **Browse and install plugins**
2. Select **pipecat-skills**
3. Select **pipecat**, **pipecat-cloud**, **pipecat-dev**, or **pipecat-mcp-server**
4. Select **Install now**

## Plugins

### pipecat

Skills for building Pipecat applications.

#### Init (`/pipecat:init`)

Scaffold a new Pipecat project with guided setup. Walks you through choosing bot type, transport, AI services, features, and deployment options, then runs `pc init` to generate the project.

```
/pipecat:init
/pipecat:init --output my-bot
```

### pipecat-cloud

Skills for deploying and managing Pipecat applications on Pipecat Cloud.

#### Deploy (`/pipecat-cloud:deploy`)

Deploy an agent to [Pipecat Cloud](https://www.pipecat.ai/cloud). Walks through the full deployment process interactively:

1. Verifies prerequisites (Pipecat Cloud CLI, Docker, authentication)
2. Creates or updates secrets from a `.env` file
3. Builds and pushes the Docker image
4. Deploys the agent

```
/pipecat-cloud:deploy
/pipecat-cloud:deploy --config examples/mybot/pcc-deploy.toml
/pipecat-cloud:deploy --config examples/mybot/pcc-deploy.toml --env examples/mybot/.env
```

Requires a `pcc-deploy.toml` configuration file in your project.

### pipecat-dev

Development workflow skills for contributing to the Pipecat project.

#### Changelog (`/pipecat-dev:changelog`)

Create changelog files for important commits in a PR. Categorizes changes by type (added, changed, fixed, deprecated, removed, security, performance) and generates files in the `changelog/` directory.

```
/pipecat-dev:changelog <pr_number>
```

#### Cleanup (`/pipecat-dev:cleanup`)

Review, refactor, and document code changes in your current branch. Analyzes uncommitted and outgoing changes for readability, performance, documentation, and consistency with Pipecat patterns.

```
/pipecat-dev:cleanup
```

#### Code Review (`/pipecat-dev:code-review`)

Automated code review for pull requests. Uses multiple specialized agents to check for bugs, logic errors, and CLAUDE.md compliance, posting inline comments on the PR.

```
/pipecat-dev:code-review <pr_number>
```

#### Docstring (`/pipecat-dev:docstring`)

Document a Python module or class using Google-style docstrings following Pipecat conventions.

```
/pipecat-dev:docstring <ClassName>
/pipecat-dev:docstring <module_path>
```

#### PR Description (`/pipecat-dev:pr-description`)

Update a GitHub PR description with a summary of changes, breaking changes, testing instructions, and linked issues.

```
/pipecat-dev:pr-description <pr_number>
/pipecat-dev:pr-description <pr_number> --fixes 123,456
```

#### PR Submit (`/pipecat-dev:pr-submit`)

Create and submit a GitHub PR from the current branch. Handles branching, committing, pushing, and automatically runs changelog and PR description generation.

```
/pipecat-dev:pr-submit
```

### pipecat-mcp-server

Skills for interacting with a [Pipecat MCP server](https://github.com/pipecat-ai/pipecat-mcp-server).

#### Talk (`/pipecat-mcp-server:talk`)

Start a voice conversation using the Pipecat MCP server. Initializes a voice agent that you can connect to via Pipecat Playground, Daily room, or phone call, and interact with using natural speech.

```
/pipecat-mcp-server:talk
```
