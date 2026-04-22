# Logs REST APIs

Two separate endpoints expose logs:

- **Pipecat Cloud**: agent/bot logs (server-side Pipecat process).
- **Daily**: call-level client logs and WebRTC metrics per participant/session.

They use different base URLs, API keys, and session ID vocabularies. Don't mix them up.

## Pipecat Cloud Logs

The CLI (`pc cloud agent logs`) wraps this endpoint. Use the REST API directly for scripting or when the CLI is unavailable.

### Endpoint

```
GET https://api.pipecat.daily.co/v1/agents/{agentName}/logs
```

### Authentication

Include your Pipecat Cloud API key in the `Authorization` header:

```
Authorization: Bearer <YOUR_API_KEY>
```

### Query Parameters

| Parameter      | Type    | Required | Description                                                        |
|---------------|---------|----------|--------------------------------------------------------------------|
| `sessionId`   | string  | No       | Filter logs to a specific session ID.                              |
| `deploymentId`| string  | No       | Filter logs to a specific deployment ID.                           |
| `query`       | string  | No       | Free-text search across log content. Useful for filtering to specific errors or keywords (e.g., `"OOMKilled"`, `"OpenAI"`). |
| `order`       | string  | No       | Sort order: `asc` or `desc`. Default `desc` (newest first).       |
| `startTime`   | string  | No       | ISO 8601 timestamp. Only return logs after this time.              |
| `endTime`     | string  | No       | ISO 8601 timestamp. Only return logs before this time.             |
| `limit`       | integer | No       | Max number of log lines to return. Default varies by implementation. |
| `offset`      | integer | No       | Number of log lines to skip (for pagination).                      |

### Example

```bash
curl -s -H "Authorization: Bearer $PCC_API_KEY" \
  "https://api.pipecat.daily.co/v1/agents/my-bot/logs?sessionId=abc-123&query=error&limit=500"
```

### Notes

- The `query` parameter supports free-text search natively, which the CLI may not expose. This is the main reason to use the REST API directly.
- For large result sets, use `limit` and `offset` for pagination.
- Logs have a retention window. Very old sessions may no longer have logs available.

## Daily Logs

Client-side call logs and WebRTC metrics per participant. Useful when a Pipecat Cloud session's agent logs look clean but the user still reported audio/connectivity issues. Daily's logs come from the browser/SDK side of the same call.

Docs: https://docs.daily.co/reference/rest-api/logs/list-logs

### Endpoint

```
GET https://api.daily.co/v1/logs
```

### Authentication

Use your **Daily** API key (not the Pipecat Cloud key). If the session ran on the Daily account paired with Pipecat Cloud, grab that key from `https://pipecat.daily.co/$ORG/settings/keys#daily`.

```
Authorization: Bearer <YOUR_DAILY_API_KEY>
```

### Query Parameters

One of `mtgSessionId` or `userSessionId` is required.

| Parameter        | Type    | Required | Description                                                                                   |
|------------------|---------|----------|-----------------------------------------------------------------------------------------------|
| `mtgSessionId`   | string  | Cond.    | Daily session ID (the call). Required if `userSessionId` not present.                         |
| `userSessionId`  | string  | Cond.    | Participant ID. Required if `mtgSessionId` not present.                                       |
| `includeLogs`    | boolean | No       | Include a `logs` array in the response. Default `true`.                                       |
| `includeMetrics` | boolean | No       | Include a `metrics` array (transport, candidate-pair, track stats). Default `false`.          |
| `logLevel`       | string  | No       | Filter by level: `ERROR`, `INFO`, or `DEBUG`.                                                 |
| `order`          | string  | No       | `ASC` or `DESC` (case-insensitive). Default `DESC`.                                           |
| `startTime`      | integer | No       | JS timestamp (ms since epoch, UTC).                                                           |
| `endTime`        | integer | No       | JS timestamp (ms since epoch). Defaults to now.                                               |
| `limit`          | integer | No       | Max records returned. Default `20`.                                                           |
| `offset`         | integer | No       | Records to skip. Default `0`.                                                                 |

### Example

```bash
curl -s -H "Authorization: Bearer $DAILY_API_KEY" \
  "https://api.daily.co/v1/logs?mtgSessionId=d1b8bd9a-bf28-4ac0-9e61-020ea0b3e27b&includeMetrics=true&logLevel=ERROR&limit=100"
```

### Notes

- Data collection starts once a second participant joins. First sample lands ~15s in, then every 15s. A single call can produce thousands of metric records.
- Default `limit` is 20; start small (e.g., `limit=100` with `logLevel=ERROR`) and page with `offset` if you need more. Pulling hundreds of unfiltered records is rarely useful.
- Pipecat Cloud session IDs (`sessionId`) are **not** the same as Daily session IDs (`mtgSessionId`). If you only have a PCC session ID, pull the Daily meeting IDs from `GET /agents/{agentName}/sessions/{sessionId}` first (the `meetingIds` array).
- Use `includeMetrics=true` when debugging connectivity or audio quality. That's where packet loss and candidate-pair stats live.
