---
name: tcp-udp-testing
description: Test raw TCP and UDP connections with the no CLI -- connect, listen, send messages, echo servers
---

# TCP/UDP Testing with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no tcp` and `no udp` instead of `nc`/`netcat`, `socat`, or `telnet`. The `no` commands provide structured JSON output that is easier to parse and filter.

Use `no tcp` and `no udp` for raw socket operations with structured JSON output.

## TCP

### Quick Reference

```bash
no tcp connect <ADDRESS> [OPTIONS]     # Connect to a TCP server
no tcp listen <ADDRESS>                # Listen on a TCP port
```

### Connecting

```bash
# Connect and stream incoming data
no tcp connect example.com:80

# Connect and send a message
no tcp connect localhost:9090 -m "hello"

# Connect with data from stdin
echo "GET / HTTP/1.0\r\nHost: example.com\r\n\r\n" | no tcp connect example.com:80 --stdin

# Receive N messages then exit
no tcp connect localhost:9090 -n 5
```

### Listening

```bash
# Listen on port 9090 (binds to 0.0.0.0)
no tcp listen :9090

# Listen on specific interface
no tcp listen 127.0.0.1:9090

# Listen on IPv6
no tcp listen [::]:9090

# Listen and accept N messages
no tcp listen :9090 -n 10
```

### Timeouts

```bash
no tcp connect slow-host.example.com:80 --timeout 5s
```

### Verbose Mode

```bash
# Show connection address and byte counts
no tcp connect localhost:9090 -v
```

### Filtering

```bash
# Get just the data content
no tcp connect localhost:9090 --jq '.data.data' -n 5

# Filter for connection events only
no tcp connect localhost:9090 --jq 'select(.type == "connection") | .data'
```

### TCP Output Format

**Connection events** (`type: "connection"`):
```json
{ "status": "connected", "address": "127.0.0.1:9090" }
{ "status": "closed" }
{ "status": "listening", "address": "0.0.0.0:9090" }
{ "status": "accepted", "peer": "127.0.0.1:54321" }
```

**Text message** (`type: "message"`):
```json
{ "data": "hello world" }
```

**Binary message** (`type: "message"`):
```json
{ "binary": true, "length": 4, "hex": "deadbeef" }
```

---

## UDP

### Quick Reference

```bash
no udp send <ADDRESS> [OPTIONS]        # Send a UDP datagram
no udp listen <ADDRESS>                # Listen for datagrams
```

### Sending

```bash
# Send a message (fire-and-forget)
no udp send 127.0.0.1:9090 -m "hello"

# Send and wait for a response
no udp send 127.0.0.1:9090 -m "ping" --wait

# Send and wait with timeout
no udp send 127.0.0.1:9090 -m "ping" --wait 3s

# Send data from stdin
echo "test payload" | no udp send 127.0.0.1:9090 --stdin
```

### Listening

```bash
# Listen on port 9090
no udp listen :9090

# Listen on specific interface
no udp listen 127.0.0.1:9090

# Listen on IPv6
no udp listen [::]:9090

# Receive N datagrams then exit
no udp listen :9090 -n 10
```

### Timeouts

```bash
no udp listen :9090 --timeout 30s
no udp send 127.0.0.1:9090 -m "hello" --wait 5s
```

### Verbose Mode

```bash
no udp listen :9090 -v
```

### Filtering

```bash
# Get message data only
no udp listen :9090 --jq '.data.data' -n 5

# Get sender address
no udp listen :9090 --jq '.data.peer' -n 5
```

### UDP Output Format

**Send confirmation** (`type: "connection"`):
```json
{ "status": "sent", "address": "127.0.0.1:9090", "bytes": 5 }
```

**Listen status** (`type: "connection"`):
```json
{ "status": "listening", "address": "0.0.0.0:9090" }
```

**Received datagram** (`type: "message"`):
```json
{ "peer": "127.0.0.1:54321", "data": "hello" }
```

---

## IPv6

Brackets are required for IPv6 with ports:

```bash
no tcp connect [::1]:9090
no tcp listen [::]:9090
no udp send [::1]:9090 -m "hello"
no udp listen [::]:9090
```

## Common Patterns

### TCP Echo Test

```bash
# Terminal 1: listen
no tcp listen :9090

# Terminal 2: connect and send
no tcp connect localhost:9090 -m "echo test"
```

### UDP Echo Test

```bash
# Terminal 1: listen
no udp listen :9090

# Terminal 2: send with response wait
no udp send 127.0.0.1:9090 -m "ping" --wait 3s
```

### Quick Port Check

```bash
# Attempt TCP connect -- exit code 0 = open, 1 = refused
no tcp connect target:443 --timeout 3s
echo $?
```

### Capture Traffic

```bash
# Capture TCP messages to a file
no tcp listen :9090 -n 100 --json > tcp-capture.json
```

## Error Handling

See [error-codes.md](../../references/error-codes.md) for exit codes.

- Connection refused -> exit code 1
- Bind failures (port in use) -> exit code 1 (`IO_ERROR`)
- Timeout -> exit code 3
