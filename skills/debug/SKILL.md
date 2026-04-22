---
name: debug
description: Debug a Pipecat Cloud agent session by analyzing logs. Accepts a local log file, pasted log content, or fetches logs remotely via the Pipecat Cloud API. Use when the user wants to debug a PCC session, analyze agent logs, or investigate a Pipecat Cloud failure.
---

Analyze logs from a Pipecat Cloud agent session and surface likely root causes. Logs can come from pasted content in the chat, a local file the user already has, or be fetched remotely via `pc cloud agent logs`.

## Step 1: Check for Pipecat Docs Access (ALWAYS do this first)

Before doing anything else, check whether the **Pipecat Docs MCP server** (`Daily Pipecat Docs` from `https://daily-docs.mcp.kapa.ai`) is available. This server provides AI-powered search over the full Pipecat and Pipecat Cloud documentation and is the best way to look up error messages, API behavior, and configuration issues found in logs.

**How to check:** Look for `mcp__claude_ai_Daily_Pipecat_Docs` tools (or `mcp__pipecat-docs__search_daily_knowledge_sources`) in your available tools. Try calling the search tool to confirm it works.

- **If available:** Use it throughout log analysis to look up any errors, exceptions, or unfamiliar behavior you find. Query it with specific error messages or symptoms before presenting your analysis.
- **If not available:** Tell the user:
  > "I don't have access to the Pipecat Docs MCP server. I'd recommend setting it up for the best debugging experience -- you can add it from https://daily-docs.mcp.kapa.ai. For now, I'll check the docs at https://docs.pipecat.ai instead."
  Then use `WebFetch` against `https://docs.pipecat.ai` to look up errors and symptoms you find in the logs.

**Do not skip this step.** Every log analysis must be informed by current Pipecat documentation, either via the MCP server or the docs site.

## Arguments

```
/debug [--file <PATH>] [--query <TEXT>] [--agent <NAME>] [--session <ID>] [--deployment <ID>] [--level <LEVEL>] [--limit <N>] [--config <PATH>]
```

- `--file` (optional): Path to a local log file. When provided, skip all remote prerequisites and fetching. Read the file directly and go straight to analysis.
- `--query` (optional): Free-text search to filter logs (remote only). Useful when the user wants to narrow to specific errors (e.g., `--query "OpenAI"` or `--query "OOMKilled"`). Passed to the API's `query` parameter.
- `--agent` (optional): Agent name (remote only). If omitted, read from the `agent_name` field of `pcc-deploy.toml` (in the current directory or at `--config`). If neither is present, ask the user.
- `--session` (optional): Session ID to filter logs for (remote only). If omitted, ask the user, or run `pc cloud agent sessions <AGENT>` to list active sessions and let them pick one.
- `--deployment` (optional): Deployment ID to filter to (from `pc cloud agent deployments <AGENT>`).
- `--level` (optional): Severity filter -- `ALL`, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. Default `ALL`.
- `--limit` (optional): Max log lines. Default `500` (CLI default is 100; override to get more context for debugging).
- `--config` (optional): Path to `pcc-deploy.toml` for agent name resolution. Defaults to `pcc-deploy.toml` in the current directory.

Examples:
- `/debug --file ~/Downloads/session-logs.txt`
- `/debug --session abc-123-def`
- `/debug --session abc-123-def --query "error"`
- `/debug --agent my-voice-bot --session abc-123-def --level ERROR`

## Choosing the Log Source

Three input modes, in priority order:

1. **Pasted content**: If the user pastes log lines directly into the chat (no `--file`, no `--session`), treat the pasted text as the log input. Skip Prerequisites, Resolving the Agent Name, Resolving the Session ID, and Fetching Logs entirely. Jump straight to **Analyzing the Logs**.

2. **Local file** (`--file` or the user mentions a path on disk): Read the file directly. Same as above -- skip all remote steps, go to analysis.

3. **Remote fetch** (no pasted content, no `--file`): Proceed with the remote sections below.

## Prerequisites (remote only)

Verify these before fetching logs. If any fails, stop and explain what to fix.

1. **Pipecat Cloud CLI**: Run `pc --version`. If missing, tell the user to install with `uv tool install pipecat-ai-cli`.
2. **Authentication**: Run `pc cloud auth whoami`. If not authenticated, run `pc cloud auth login --headless` as a background task, extract the URL and six-digit code from the output file, and share both with the user. Wait for completion before proceeding.

## Resolving the Agent Name (remote only)

If `--agent` is not provided:
1. Look for `pcc-deploy.toml` at `--config` or in the current directory. Parse it and read `agent_name`.
2. If no config file exists, ask the user for the agent name.

## Resolving the Session ID (remote only)

If `--session` is not provided:
1. Ask the user if they have a session ID. If yes, use it.
2. If no, run `pc cloud agent sessions <AGENT>` to list active sessions and show the output. Ask the user to pick one.

## Fetching Logs (remote only)

Run from any directory (session logs are not tied to the config directory):

```
pc cloud agent logs <AGENT> --session-id <SESSION_ID> [--level <LEVEL>] [--limit <N>] [--deployment <DEPLOYMENT_ID>]
```

If `--query` was provided, the CLI may not support free-text search directly. In that case, fetch the logs and filter locally with `grep` or equivalent. Alternatively, use the REST API endpoint which supports the `query` parameter natively (see `references/rest-api.md`).

Capture the output. If the command returns no logs:
- Confirm the session ID is correct (typos, wrong agent).
- Pipecat Cloud sessions have a retention window -- very old sessions may no longer have logs. Note this to the user.
- Some pods have log-exclusion annotations and may not emit logs. If the session clearly ran but no logs come back, tell the user this is likely the case.

## Handling Large Logs

Logs can be large, especially with high `--limit` values or big local files. If the logs exceed roughly 500 lines:

1. **Filter first**: Scan for lines containing `ERROR`, `WARNING`, `CRITICAL`, `Traceback`, `Exception`, or `OOM`. Present this filtered view to the user.
2. **Summarize the rest**: Note how many total lines there are, the time range they cover, and whether the non-error lines look routine (startup, heartbeat, shutdown).
3. **Offer the full output**: Ask the user if they want to see the unfiltered logs or a specific time window.

This keeps analysis focused and avoids flooding the conversation with routine log lines.

## Analyzing the Logs

Analysis is docs-first. The pattern list below is a fallback, not the primary method.

1. **Identify key errors**: Scan the logs for exceptions, error messages, stack traces, and unexpected shutdowns.
2. **Query the docs**: For each distinct error or symptom, query the Pipecat Docs MCP server (or `WebFetch` on `https://docs.pipecat.ai`) with the specific error message. The docs may describe the exact cause, a known bug, or a recommended fix. This is the most reliable way to diagnose issues because the docs reflect current Pipecat behavior.
3. **Fall back to pattern matching**: If the docs don't surface a match, check against these common patterns:
   - **Cold start / image pull failures**: `ImagePullBackOff`, `ErrImagePull`, long gaps before "Application startup complete". Cross-region ECR pulls can cause this.
   - **OOM / resource limits**: `OOMKilled`, `Memory cgroup out of memory`, process termination without clean shutdown.
   - **Transport / WebRTC errors**: Daily transport failures, ICE connection errors, `CGNAT`-related signals.
   - **LLM / STT / TTS provider errors**: Auth failures, rate limits, timeouts from OpenAI / Cartesia / Deepgram / etc.
   - **Bot function errors**: Unhandled exceptions, traceback blocks, `await` errors in the pipeline.
   - **Graceful shutdown vs crash**: `Application shutdown complete` is clean; missing shutdown logs after a client disconnect suggests a crash.

Surface 2-4 candidate root causes ranked by likelihood, each with: the matching log line(s), what it typically means, and a next step.

## Debugging Calls: Audio Not Coming Through

Check the Pipecat logs first. If the agent logs look clean but the user reports an audio issue (nothing coming through, one-way audio, robotic/choppy audio) and the call uses Daily WebRTC transport, follow up by pulling Daily's call logs. The Pipecat agent sees its own process; Daily sees the transport layer and the participant side.

1. Grab the Daily session ID (`mtgSessionId`) for the call. If you only have a Pipecat Cloud session ID, run `pc cloud agent sessions <AGENT> --id <SESSION_ID>` and read the Daily meeting IDs from the `meetingIds` array.
2. Hit `https://api.daily.co/v1/logs` with `DAILY_API_KEY` as a bearer token. Include `includeMetrics=true` for packet loss and candidate-pair stats.

```bash
curl -s -H "Authorization: Bearer $DAILY_API_KEY" \
  "https://api.daily.co/v1/logs?mtgSessionId=<DAILY_SESSION_ID>&includeMetrics=true&limit=500"
```

The Daily API key for a Pipecat Cloud project lives at `https://pipecat.daily.co/$ORG/settings/keys#daily`. See `references/rest-api.md` for the full parameter list.

## REST API

The CLI wraps a REST endpoint. For details on using it directly (e.g., for scripting or when the CLI is unavailable), see `references/rest-api.md`. That file also documents Daily's `/logs` endpoint for call-side debugging.

## Completion

After presenting the analysis, ask the user whether they want to:
- Fetch more logs (wider time range, higher limit, different level, or a `--query` filter)
- Pull logs for a different session on the same agent
- Package the findings as a debug report (see template below)

## Debug Report Template

When the user asks to package the findings, first fetch session metadata if a remote session is available:

```
pc cloud agent sessions <AGENT> --id <SESSION_ID>
```

This returns timing, completion status, cold-start duration, and CPU/memory percentiles. Use these to fill in the report template. If the user provided a local file or pasted content without a session ID, mark metadata fields as "N/A".

Then produce a report using this template. Fill in every field from the data you have. If a field is unavailable, write "N/A" rather than omitting it.

```markdown
## PCC Session Debug: {session_id}

**Status**: {Resolved | Investigating | Needs escalation}
**Agent**: {agentName}
**Deployment**: {deploymentId}
**Time window**: {createdAt} -- {endedAt}
**Completion status**: {completionStatus}
**Cold start**: {yes/no} (botStartSeconds: {n})

### Summary
{1-2 sentence description of what happened}

### Log Timeline
- {timestamp}: {key event}
- {timestamp}: {key event}
- ...

### Resource Analysis
- Peak memory: {X}MB / {limit}MB ({percentage}%)
- CPU P99: {cpuMillicoresP99} millicores
- Trend: {stable | climbing | spike at init | sawtooth}

### Root Cause
{Diagnosis based on correlated logs + metrics + docs findings}

### Matching Known Issues
{Results from Pipecat Docs MCP server or docs site search, with links. Or "No matching known issues."}

### Recommended Next Steps
1. {Specific action}
2. {Specific action}
3. {Specific action}
```

### Filling in the template

- **Status**: Use "Resolved" if you identified a clear fix, "Investigating" if the cause is uncertain, "Needs escalation" if it requires action from Daily support or the Pipecat team.
- **Time window / Completion status / Cold start**: Pull from `pc cloud agent sessions <AGENT> --id <SESSION_ID>`. If unavailable (local file, no session ID), mark as "N/A".
- **Log Timeline**: Extract the 5-10 most important events in chronological order. Focus on state transitions (startup, connection, errors, shutdown), not routine log lines.
- **Resource Analysis**: Populate from session metadata (CPU/memory percentiles). If unavailable, write "No metrics available -- run `pc cloud agent sessions <AGENT> --id <SESSION_ID>` to check CPU/memory percentiles."
- **Matching Known Issues**: Query the Pipecat Docs MCP server (or docs site) with the root cause symptoms. Include any matching docs links. If nothing matches, say so explicitly.
- **Recommended Next Steps**: Be specific. Name the exact config change, CLI command, or where to escalate (e.g., "Post in the Pipecat Discord #pipecat-help channel with session ID and log excerpts", "Contact Daily support at help@daily.co"). Avoid generic advice like "check the logs."
