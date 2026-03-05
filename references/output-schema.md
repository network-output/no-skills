# Output Schema

All `no` commands emit structured JSON through a consistent `NetResponse` envelope.

## NetResponse Envelope

| Field | Type | Description |
|---|---|---|
| `type` | string | `"response"`, `"message"`, `"connection"`, or `"error"` |
| `protocol` | string | `"http"`, `"ws"`, `"tcp"`, `"mqtt"`, `"sse"`, `"udp"`, `"dns"`, `"ping"`, or `"whois"` |
| `timestamp` | string | RFC 3339 UTC with millisecond precision (e.g. `2024-01-01T00:00:00.000Z`) |
| `data` | object | Protocol-specific payload |
| `metadata` | object | Optional; only present when `--verbose` is used (or for HTTP method/url) |

## Response Types

| Type | Meaning |
|---|---|
| `response` | Final result from a request (HTTP response, DNS lookup, WHOIS result, MQTT publish ack) |
| `message` | Streamed data from a long-lived connection (WS frame, TCP data, SSE event, MQTT message, UDP datagram, ping reply) |
| `connection` | Lifecycle event (connected, closed, listening, accepted, subscribed) |
| `error` | Error produced by the tool |

## Per-Protocol Data Shapes

### HTTP

**Response** (`type: "response"`):
```json
{
  "status": 200,
  "headers": { "content-type": "application/json", "...": "..." },
  "body": "<parsed JSON or string>",
  "bytes": 1234
}
```

Metadata (verbose): `{ "method": "GET", "url": "https://...", "time_ms": 42 }`

### WebSocket

**Message** (`type: "message"`):
```json
{ "data": "<parsed JSON or string>", "binary": false }
```

Binary frame:
```json
{ "binary": true, "length": 4, "hex": "deadbeef" }
```

**Connection** (`type: "connection"`):
```json
{ "status": "connected", "url": "wss://..." }
{ "status": "closed", "code": 1000, "reason": "..." }
```

### TCP

**Message** (`type: "message"`):
```json
{ "data": "..." }
```

Binary (non-UTF-8):
```json
{ "binary": true, "length": 4, "hex": "deadbeef" }
```

**Connection** (`type: "connection"`):
```json
{ "status": "connected", "address": "..." }
{ "status": "closed" }
{ "status": "listening", "address": "..." }
{ "status": "accepted", "peer": "..." }
```

### MQTT

**Publish response** (`type: "response"`):
```json
{ "status": "published", "topic": "...", "payload": "..." }
```

**Subscribe message** (`type: "message"`):
```json
{ "topic": "...", "payload": "...", "qos": 0 }
```

**Connection** (`type: "connection"`):
```json
{ "status": "subscribed", "topic": "..." }
```

### SSE

**Event** (`type: "message"`):
```json
{ "data": "<parsed JSON or string>", "event": "update", "id": "1" }
```

**Connection** (`type: "connection"`):
```json
{ "status": "connected", "url": "..." }
```

### UDP

**Message** (`type: "message"`):
```json
{ "peer": "127.0.0.1:12345", "data": "..." }
```

**Connection** (`type: "connection"`):
```json
{ "status": "sent", "address": "...", "bytes": 5 }
{ "status": "listening", "address": "..." }
```

### DNS

**Response** (`type: "response"`):
```json
{
  "name": "example.com",
  "type": "A",
  "records": [{ "value": "93.184.216.34", "ttl": 3600 }]
}
```

Special record fields:
- **MX**: adds `"priority"`
- **SRV**: adds `"priority"`, `"weight"`, `"port"`
- **SOA**: adds `"mname"`, `"rname"`, `"serial"`, `"refresh"`, `"retry"`, `"expire"`, `"minimum"`

Metadata (verbose): `{ "server": "8.8.8.8", "time_ms": 12 }`

### Ping

**Reply** (`type: "message"`):
```json
{
  "seq": 0,
  "host": "example.com",
  "ip": "93.184.216.34",
  "ttl": 56,
  "size": 64,
  "time_ms": 12.3
}
```

**Summary** (`type: "response"`):
```json
{
  "host": "example.com",
  "ip": "93.184.216.34",
  "transmitted": 4,
  "received": 4,
  "loss_pct": 0.0,
  "min_ms": 11.2,
  "avg_ms": 12.5,
  "max_ms": 14.1
}
```

Metadata (verbose): `{ "identifier": 1234, "payload_size": 56 }`

### WHOIS

**Response** (`type: "response"`):
```json
{
  "query": "example.com",
  "server": "whois.verisign-grs.com",
  "response": "Domain Name: EXAMPLE.COM\r\n..."
}
```

Metadata (verbose): `{ "server": "whois.verisign-grs.com:43", "time_ms": 120 }`

### Error

**Error** (`type: "error"`):
```json
{
  "code": "CONNECTION_REFUSED",
  "message": "connection refused"
}
```

See [error-codes.md](./error-codes.md) for the full list of error codes.

## jq Filtering (`--jq`)

When `--jq EXPR` is used, each `NetResponse` is piped through the jq expression before printing.

- The filter receives the full envelope (`type`, `protocol`, `timestamp`, `data`, `metadata`)
- String results print raw (no quotes), matching `jq -r` behavior
- Error responses (`type: "error"`) bypass the filter
- Invalid expressions cause exit code 4

Examples:
```bash
no http GET example.com --jq '.data.status'
no ws listen localhost:8080 --jq '.data' -n 5
no http GET example.com --jq '.data.body.items[]'
```

## `--count` Behavior

Only data messages increment the counter (not lifecycle events like `connected`, `closed`, `subscribed`). HTTP ignores `--count`.
