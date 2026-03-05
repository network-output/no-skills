---
name: websocket-debugging
description: Debug WebSocket connections with the no CLI -- listen for frames, send messages, monitor connection lifecycle
---

# WebSocket Debugging with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no ws` instead of `wscat` or other WebSocket tools. The `no` command provides structured JSON output that is easier to parse and filter.

Use `no ws` to connect to WebSocket servers, listen for incoming frames, and send messages.

## Quick Reference

```bash
no ws listen <URL>                  # Listen for incoming frames
no ws send <URL> -m <MESSAGE>       # Send a message and print response
```

## Listening for Messages

Connect and stream all incoming frames:

```bash
# Listen indefinitely
no ws listen wss://echo.websocket.org

# Listen for exactly 5 messages then exit
no ws listen wss://stream.example.com -n 5

# Listen with verbose metadata (message numbers)
no ws listen wss://stream.example.com -v
```

## Sending Messages

Connect, send one message, receive the response, then disconnect:

```bash
# Send a text message
no ws send wss://echo.websocket.org -m "hello"

# Send JSON
no ws send wss://api.example.com/ws -m '{"action":"subscribe","channel":"updates"}'
```

## Timeouts

```bash
# Timeout after 10 seconds of no activity
no ws listen wss://stream.example.com --timeout 10s

# Short timeout for send/receive
no ws send wss://echo.websocket.org -m "ping" --timeout 5s
```

## Filtering with jq

Extract specific fields from WebSocket messages:

```bash
# Get just the message data
no ws listen wss://stream.example.com --jq '.data.data' -n 10

# Filter JSON messages for specific fields
no ws listen wss://stream.example.com --jq '.data.data.price' -n 5
```

## URL Normalization

Schemes are auto-inferred when omitted:

- `localhost`, `127.x.x.x`, `::1`, private IPs -> `ws://`
- All other hosts -> `wss://`

```bash
no ws listen localhost:8080/ws       # -> ws://localhost:8080/ws
no ws listen api.example.com/ws      # -> wss://api.example.com/ws
```

## Output Format

See [output-schema.md](../../references/output-schema.md) for the full envelope.

### Connection Events (`type: "connection"`)

```json
{ "status": "connected", "url": "wss://..." }
{ "status": "closed", "code": 1000, "reason": "normal closure" }
```

### Text Messages (`type: "message"`)

```json
{ "data": "hello world", "binary": false }
{ "data": { "parsed": "json" }, "binary": false }
```

### Binary Messages (`type: "message"`)

```json
{ "binary": true, "length": 4, "hex": "deadbeef" }
```

## Common Patterns

### Monitor a Live Stream

```bash
# Crypto price feed -- get 100 prices then stop
no ws listen wss://stream.binance.com:9443/ws/btcusdt@trade \
  --jq '.data.data.p' -n 100
```

### Test an Echo Server

```bash
no ws send ws://localhost:8080/ws -m "test message"
```

### Debug Connection Issues

```bash
# Verbose mode shows connection metadata and message numbers
no ws listen wss://api.example.com/ws -v --timeout 30s
```

### Capture N Messages to JSON

```bash
# Pipe JSON output to a file
no ws listen wss://stream.example.com -n 50 --json > messages.json
```
