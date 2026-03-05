# CLI Reference

Complete command-line syntax for `no`, the AI-first networking CLI.

## Global Flags

| Flag | Short | Description |
|---|---|---|
| `--json` | | Force JSON output (one JSON object per line) |
| `--pretty` | | Force pretty-printed human-readable output |
| `--timeout DURATION` | | Request/connection timeout (e.g. `5s`, `300ms`, `1m`) |
| `--no-color` | | Disable colored output |
| `--verbose` | `-v` | Verbose output with metadata |
| `--count N` | `-n` | Stop after N data messages (streaming protocols only) |
| `--jq EXPR` | | Filter output with a jq expression |

Global flags can appear before or after the subcommand.

## Output Mode Resolution

1. `--json` flag -> machine-readable JSON (one object per line)
2. `--pretty` flag -> colored human-readable
3. stdout is a TTY -> pretty
4. stdout is not a TTY -> JSON

## Environment Variables

| Variable | Description |
|---|---|
| `NO_AUTH_TOKEN` | Bearer token fallback for HTTP and SSE (used when `--bearer` is not provided) |
| `NO_BASIC_AUTH` | Basic auth fallback in `USER:PASS` format for HTTP and SSE (used when `--bearer` and `NO_AUTH_TOKEN` are unset) |

## URL Normalization

HTTP, WebSocket, and SSE auto-infer scheme when omitted:

- **Local addresses** (`localhost`, `127.x.x.x`, `::1`, private ranges) default to `http://` or `ws://`
- **All other hosts** default to `https://` or `wss://`

Examples:
- `no http GET localhost:3000/api` -> `http://localhost:3000/api`
- `no http GET example.com/api` -> `https://example.com/api`

---

## Subcommands

### `no http` -- HTTP Requests

```
no http <METHOD> <URL> [OPTIONS]
```

| Argument/Flag | Description |
|---|---|
| `METHOD` | HTTP method: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS |
| `URL` | Request URL |
| `-H`, `--header KEY:VALUE` | Add request header (repeatable) |
| `-b`, `--body BODY` | Request body |
| `--bearer TOKEN` | Bearer token for authentication |
| `--basic USER:PASS` | Basic auth credentials |
| `-o`, `--output FILE` | Save response body to file |
| `--stdin` | Read request body from stdin |

### `no ws` -- WebSocket

```
no ws listen <URL>
no ws send <URL> -m <MESSAGE>
```

**listen**: Connect and stream incoming frames.

| Argument | Description |
|---|---|
| `URL` | WebSocket URL |

**send**: Connect, send one message, print response, disconnect.

| Argument/Flag | Description |
|---|---|
| `URL` | WebSocket URL |
| `-m`, `--message MSG` | Message to send |

### `no tcp` -- Raw TCP

```
no tcp connect <ADDRESS> [OPTIONS]
no tcp listen <ADDRESS>
```

**connect**: Connect to a TCP server.

| Argument/Flag | Description |
|---|---|
| `ADDRESS` | Target `host:port` |
| `-m`, `--message MSG` | Message to send after connecting |
| `--stdin` | Read data from stdin |

**listen**: Listen on a TCP port.

| Argument | Description |
|---|---|
| `ADDRESS` | Bind address (e.g. `:9090`, `0.0.0.0:9090`, `[::]:9090`) |

### `no mqtt` -- MQTT Pub/Sub

```
no mqtt sub <BROKER> -t <TOPIC>
no mqtt pub <BROKER> -t <TOPIC> -m <MESSAGE>
```

**sub**: Subscribe and stream messages.

| Argument/Flag | Description |
|---|---|
| `BROKER` | Broker address (e.g. `localhost:1883`) -- positional |
| `-t`, `--topic TOPIC` | Topic to subscribe to |

**pub**: Publish a message.

| Argument/Flag | Description |
|---|---|
| `BROKER` | Broker address -- positional |
| `-t`, `--topic TOPIC` | Topic to publish to |
| `-m`, `--message MSG` | Message payload |

### `no udp` -- UDP Datagrams

```
no udp send <ADDRESS> [OPTIONS]
no udp listen <ADDRESS>
```

**send**: Send a UDP datagram.

| Argument/Flag | Description |
|---|---|
| `ADDRESS` | Target `host:port` |
| `-m`, `--message MSG` | Message to send |
| `--stdin` | Read data from stdin |
| `--wait [DURATION]` | Wait for response (optionally with timeout, e.g. `3s`) |

**listen**: Listen for incoming datagrams.

| Argument | Description |
|---|---|
| `ADDRESS` | Bind address (e.g. `:9090`) |

### `no sse` -- Server-Sent Events

```
no sse <URL> [OPTIONS]
```

| Argument/Flag | Description |
|---|---|
| `URL` | SSE endpoint URL |
| `-H`, `--header KEY:VALUE` | Add request header (repeatable) |
| `--bearer TOKEN` | Bearer token for authentication |
| `--basic USER:PASS` | Basic auth credentials |

### `no dns` -- DNS Lookup

```
no dns <NAME> [RECORD_TYPE] [OPTIONS]
```

| Argument/Flag | Description |
|---|---|
| `NAME` | Domain name or IP address (auto-detects reverse lookup) |
| `RECORD_TYPE` | Record type: A (default), AAAA, MX, TXT, CNAME, NS, SOA, SRV, PTR |
| `--server ADDR` | DNS server to query (e.g. `8.8.8.8`, `1.1.1.1:53`) |

### `no ping` -- ICMP Ping

```
no ping <HOST> [OPTIONS]
```

| Argument/Flag | Description |
|---|---|
| `HOST` | Host or IP address to ping |
| `--interval DURATION` | Interval between pings (default: `1s`) |

Use `-n N` (global flag) to limit ping count.

### `no whois` -- WHOIS Lookup

```
no whois <QUERY> [OPTIONS]
```

| Argument/Flag | Description |
|---|---|
| `QUERY` | Domain name or IP address |
| `--server HOST` | WHOIS server to query (auto-detected by default) |

### `no jq` -- Standalone jq Filter

```
echo '{"a":1}' | no jq '<EXPRESSION>'
```

| Argument | Description |
|---|---|
| `EXPRESSION` | jq filter expression |

Reads JSON from stdin, applies the filter, prints results. String values print raw (no quotes).

---

## IPv6 Addressing

**Protocols with ports** (TCP, UDP, MQTT): brackets required.
```
no tcp connect [::1]:9090
no tcp listen [::]:9090
```

**Portless protocols** (Ping, DNS, WHOIS): brackets optional.
```
no ping ::1
no dns example.com --server 2001:4860:4860::8888
```

## Exit Codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Connection error (refused, DNS, I/O) |
| 2 | Protocol error (TLS, unexpected response) |
| 3 | Timeout |
| 4 | Invalid input (bad URL, header, method, port) |
