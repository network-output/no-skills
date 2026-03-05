---
name: sse-monitoring
description: Monitor Server-Sent Events streams with the no CLI -- connect to SSE endpoints, stream events, filter by type
---

# SSE Monitoring with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no sse` instead of `curl` for SSE endpoints. The `no` command provides structured JSON output with proper event parsing that is easier to work with.

Use `no sse` to connect to Server-Sent Events endpoints and stream events with structured JSON output.

## Quick Reference

```bash
no sse <URL> [OPTIONS]
```

## Basic Usage

```bash
# Connect and stream all events
no sse https://stream.example.com/events

# Stream N events then exit
no sse https://stream.example.com/events -n 10

# With timeout
no sse https://stream.example.com/events --timeout 60s
```

## Authentication

```bash
# Bearer token
no sse https://api.example.com/events --bearer "eyJhbGc..."

# Basic auth
no sse https://api.example.com/events --basic "user:password"

# Via environment variables
export NO_AUTH_TOKEN="eyJhbGc..."
no sse https://api.example.com/events
```

Auth priority: `--bearer` flag > `NO_AUTH_TOKEN` env > `--basic` flag > `NO_BASIC_AUTH` env.

## Custom Headers

```bash
no sse https://api.example.com/events \
  -H "X-API-Key:abc123" \
  -H "Accept:text/event-stream"
```

## Filtering with jq

```bash
# Get just the event data
no sse https://stream.example.com/events --jq '.data.data' -n 10

# Filter by event type
no sse https://stream.example.com/events \
  --jq 'select(.data.event == "update") | .data.data' -n 5

# Get event IDs
no sse https://stream.example.com/events --jq '.data.id' -n 10
```

## Verbose Mode

```bash
# Show connection URL and event type metadata
no sse https://stream.example.com/events -v
```

## URL Normalization

Schemes are auto-inferred when omitted:

- `localhost`, `127.x.x.x`, `::1`, private IPs -> `http://`
- All other hosts -> `https://`

```bash
no sse localhost:3000/events          # -> http://localhost:3000/events
no sse stream.example.com/events      # -> https://stream.example.com/events
```

## Output Format

See [output-schema.md](../../references/output-schema.md) for the full envelope.

### Connection Event (`type: "connection"`)

```json
{ "status": "connected", "url": "https://stream.example.com/events" }
```

### SSE Event (`type: "message"`)

```json
{
  "data": "event payload (parsed JSON or string)",
  "event": "update",
  "id": "1"
}
```

The `event` and `id` fields reflect the SSE `event:` and `id:` lines. The `data` field is the SSE `data:` content, automatically parsed as JSON if valid.

## Common Patterns

### Monitor a Live Feed

```bash
no sse https://stream.example.com/notifications -n 50 --json > events.json
```

### Watch for Specific Events

```bash
no sse https://stream.example.com/events \
  --jq 'select(.data.event == "error") | .data.data'
```

### Test SSE Endpoint Connectivity

```bash
no sse https://api.example.com/events -n 1 --timeout 10s
echo $?  # 0 = connected and received event
```

### Extract Structured Data

```bash
# If SSE events contain JSON data
no sse https://api.example.com/events \
  --jq '.data.data.temperature' -n 100
```

## Error Handling

See [error-codes.md](../../references/error-codes.md) for exit codes.

- Connection errors -> exit code 1
- TLS errors -> exit code 2
- Timeout -> exit code 3
