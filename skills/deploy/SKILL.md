---
name: deploy
description: Deploy an agent image to Pipecat Cloud
---

Deploy an agent to Pipecat Cloud. This skill walks through the full deployment process interactively, confirming each step with the user.

## Arguments

```
/deploy [--config <PATH>] [--env <PATH>]
```

- `--config` (optional): Path to `pcc-deploy.toml`. Defaults to `pcc-deploy.toml` in the current directory.
- `--env` (optional): Path to `.env` file for creating secrets. If not provided, the skill will look for a `.env` file in the same directory as the config file.

Examples:
- `/deploy`
- `/deploy --config examples/mybot/pcc-deploy.toml`
- `/deploy --config examples/mybot/pcc-deploy.toml --env examples/mybot/.env`

## Prerequisites

Before starting, verify these prerequisites and inform the user about any that are missing:

1. **Pipecat Cloud CLI**: Check if `pc` is installed by running `pc --version`. If not installed, tell the user to install it with `uv tool install pipecat-ai-cli` and stop.

2. **Pipecat Cloud authentication**: Check if `pc cloud auth whoami` succeeds. If not authenticated, run `pc cloud auth login --headless` as a **background task**, then read the output file to extract the URL and six-digit code. Share both with the user so they can authenticate in their browser. Wait for the background task to complete before proceeding.

3. **Docker**: Check if `docker info` succeeds (daemon running). If not, tell the user to start Docker and stop. Also check `docker login` succeeds. If not logged in, tell the user to run `docker login` and stop.

4. **Configuration file**: Read the `pcc-deploy.toml` file (from `--config` argument or current directory). If it doesn't exist, tell the user they need a `pcc-deploy.toml` and stop. Parse and display the configuration to the user (agent_name, image, secret_set, etc.).

If any prerequisite fails, stop and clearly explain what needs to be fixed. Do not proceed to the next steps.

## Important: Running Commands

All `pc cloud` commands must be run from the directory containing the `pcc-deploy.toml` file. Use `cd <dir> && <command>` to ensure the correct working directory.

Several `pc cloud` commands prompt for interactive confirmation which doesn't work in this environment. Use these patterns:
- **`pc cloud docker build-push`**: Pipe `yes |` to auto-confirm (e.g., `yes | pc cloud docker build-push`)
- **`pc cloud deploy`**: Pipe `yes |` to auto-confirm (e.g., `yes | pc cloud deploy`)
- **`pc cloud secrets set`**: Use the `--skip` flag to skip confirmation (e.g., `pc cloud secrets set NAME --file .env --skip`)

## Deployment Steps

Walk through each step interactively, asking the user to confirm before proceeding.

### Step 1: Secrets Setup

Ask the user if they need to create or update secrets for this deployment.

- If yes, determine the env file path (from `--env` argument, or look for `.env` in the same directory as the config file, or ask the user).
- Read the `secret_set` name from the `pcc-deploy.toml` configuration.
- Run: `pc cloud secrets set {SECRET_SET_NAME} --file {ENV_FILE_PATH} --skip`
- Show the output to the user.

- If no, skip this step.

### Step 2: Build and Push Docker Image

Ask the user if they want to build and push the Docker image.

- If yes:
  - First, check if `uv.lock` exists in the config directory. If so, run `uv lock` to ensure it's up to date before building.
  - Run from the config directory: `yes | pc cloud docker build-push`
  - Use a generous timeout (5 minutes) as builds can take a while.
  - If the build fails due to a stale lockfile, run `uv lock` in the config directory and retry.
  - If the build fails for other reasons, show the error and ask the user how to proceed.

- If no, skip this step (image may already be pushed).

### Step 3: Deploy

Ask the user to confirm they want to deploy the agent.

- Show a summary of what will be deployed (agent_name, image, secret_set from the config).
- Run from the config directory: `yes | pc cloud deploy`
- Use a generous timeout (5 minutes) as deployment may take time to reach ready state.
- If deployment times out but no error occurred, check logs with `pc cloud agent logs {AGENT_NAME}` and share with the user — the deployment may still be starting up.
- Show the deployment output and status to the user.

## Error Handling

- If any `pc` command fails, show the full error output and explain what might have gone wrong.
- Common issues to check for:
  - Docker not logged in to the image registry (`docker login`)
  - Invalid or expired Pipecat Cloud authentication
  - Missing or malformed `pcc-deploy.toml`
  - Secret set name mismatch between config and what exists in Pipecat Cloud
  - Stale `uv.lock` file — run `uv lock` to fix

## Completion

After a successful deployment, summarize what was done:
- Secrets created/updated (if applicable)
- Image built and pushed (if applicable)
- Agent deployed with name and image from config
