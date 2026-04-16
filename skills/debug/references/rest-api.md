# Pipecat Cloud Logs REST API

The CLI (`pc cloud agent logs`) wraps this endpoint. Use the REST API directly for scripting or when the CLI is unavailable.

## Endpoint

```
GET https://api.pipecat.daily.co/v1/agents/{agentName}/logs
```

## Authentication

Include your Pipecat Cloud API key in the `Authorization` header:

```
Authorization: Bearer <YOUR_API_KEY>
```

## Query Parameters

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

## Example

```bash
curl -s -H "Authorization: Bearer $PCC_API_KEY" \
  "https://api.pipecat.daily.co/v1/agents/my-bot/logs?sessionId=abc-123&query=error&limit=500"
```

## Notes

- The `query` parameter supports free-text search natively, which the CLI may not expose. This is the main reason to use the REST API directly.
- For large result sets, use `limit` and `offset` for pagination.
- Logs have a retention window. Very old sessions may no longer have logs available.
