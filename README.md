# Pipecat Skills for Claude Code

[Skills](https://docs.anthropic.com/en/docs/claude-code/skills) for building and deploying [Pipecat](https://www.pipecat.ai/) applications with Claude Code.

## Installation

Register this repository as a Claude Code Plugin marketplace:

```
/plugin marketplace add pipecat-ai/skills
```

Then install the plugin:

```
/plugin install pipecat-cloud@pipecat-skills
```

Or browse and install interactively:

1. Select **Browse and install plugins**
2. Select **pipecat-skills**
3. Select **pipecat-cloud**
4. Select **Install now**

## Skills

### Deploy (`/pipecat-cloud:deploy`)

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
