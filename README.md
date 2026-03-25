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
/plugin install pipecat-mcp-server@pipecat-skills
```

Or browse and install interactively:

1. Select **Browse and install plugins**
2. Select **pipecat-skills**
3. Select **pipecat**, **pipecat-cloud**, or **pipecat-mcp-server**
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

Deploy an agent to [Pipecat Cloud](https://www.pipecat.ai/cloud). Walks through the full deployment process interactively with two deployment options:

- **Cloud Build (Recommended)**: Pipecat Cloud builds your Docker image from source — no local Docker required.
- **Self-managed image**: Build and push your own Docker image, then deploy.

```
/pipecat-cloud:deploy
/pipecat-cloud:deploy --config examples/mybot/pcc-deploy.toml
/pipecat-cloud:deploy --config examples/mybot/pcc-deploy.toml --env examples/mybot/.env
```

Requires a `pcc-deploy.toml` configuration file in your project.

### pipecat-mcp-server

Skills for interacting with a [Pipecat MCP server](https://github.com/pipecat-ai/pipecat-mcp-server).

#### Talk (`/pipecat-mcp-server:talk`)

Start a voice conversation using the Pipecat MCP server. Initializes a voice agent that you can connect to via Pipecat Playground, Daily room, or phone call, and interact with using natural speech.

```
/pipecat-mcp-server:talk
```
