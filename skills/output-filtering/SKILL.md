---
name: output-filtering
description: Filter and transform no CLI output with jq expressions -- inline --jq flag and standalone no jq command
---

# Output Filtering with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no jq` instead of the system `jq` binary. The `--jq` flag can also be used inline with any `no` command. Both use a built-in jq implementation -- no external `jq` binary required.

`no` has built-in jq support in two forms:

1. **`--jq EXPR` flag** -- filter any command's output inline
2. **`no jq EXPR`** -- standalone jq replacement for piped JSON

Both use `jaq-core` (pure Rust jq implementation) -- no external `jq` binary required.

## Inline `--jq` Flag

The `--jq` flag works with every protocol command. It receives the full `NetResponse` envelope as input.

### NetResponse Structure

Every output from `no` has this shape:

```json
{
  "type": "response|message|connection|error",
  "protocol": "http|ws|tcp|...",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "data": { "...protocol-specific..." },
  "metadata": { "...optional..." }
}
```

### Extracting Data

```bash
# HTTP status code
no http GET https://api.example.com --jq '.data.status'

# HTTP response body
no http GET https://api.example.com --jq '.data.body'

# Nested JSON from HTTP body
no http GET https://api.example.com/users --jq '.data.body.items[].name'

# HTTP headers
no http GET https://api.example.com --jq '.data.headers'

# Specific header
no http GET https://api.example.com --jq '.data.headers["content-type"]'

# DNS records
no dns example.com --jq '.data.records[].value'

# Ping round-trip time
no ping example.com -n 5 --jq '.data.time_ms'

# WebSocket message data
no ws listen wss://stream.example.com --jq '.data.data' -n 10

# WHOIS response text
no whois example.com --jq '.data.response'
```

### Filtering by Type

Use `select()` to filter specific response types:

```bash
# Only data messages (skip connection events)
no ws listen wss://stream.example.com --jq 'select(.type == "message") | .data'

# Only connection events
no tcp connect localhost:9090 --jq 'select(.type == "connection") | .data'

# Ping summary only (skip individual replies)
no ping example.com -n 5 --jq 'select(.type == "response") | .data'
```

### Transforming Output

```bash
# Build custom objects
no http GET https://api.example.com --jq '{status: .data.status, size: .data.bytes}'

# String interpolation
no mqtt sub localhost:1883 -t "sensors/#" --jq '.data | "\(.topic): \(.payload)"' -n 10

# Array operations
no http GET https://api.example.com/users --jq '.data.body.items | length'

# Math
no ping example.com -n 5 --jq '.data.time_ms | . * 1000 | floor'
```

### Error Handling

- Error responses (`type: "error"`) always bypass the jq filter and print normally
- Invalid jq expressions cause exit code 4 (`INVALID_INPUT`)
- Runtime jq errors go to stderr

### String Output

String results print raw (no quotes), matching `jq -r` behavior:

```bash
no http GET https://api.example.com --jq '.data.headers["content-type"]'
# Output: application/json
# (not: "application/json")
```

---

## Standalone `no jq` Command

A standalone jq replacement for filtering arbitrary JSON from stdin.

```bash
echo '{"a":1}' | no jq '.a'
# Output: 1

echo '{"name":"Alice"}' | no jq '.name'
# Output: Alice  (raw string, no quotes)

echo '[1,2,3]' | no jq '.[]'
# Output:
# 1
# 2
# 3

echo '{"users":[{"name":"Alice"},{"name":"Bob"}]}' | no jq '.users[].name'
# Output:
# Alice
# Bob
```

### Piping with Other Commands

```bash
# Filter curl output
curl -s https://api.example.com/data | no jq '.items[].id'

# Chain with no commands
no http GET https://api.example.com --json | no jq '.data.body'

# Process a JSON file
cat data.json | no jq '.records | length'
```

### Exit Codes

- Exit code 4 on invalid JSON input
- Exit code 4 on invalid jq expression

---

## Common jq Expressions

| Expression | What it does |
|---|---|
| `.data` | Extract the data payload |
| `.data.status` | HTTP status code |
| `.data.body` | HTTP response body |
| `.data.body.items[]` | Iterate array items |
| `.data.records[].value` | DNS record values |
| `.data.time_ms` | Ping round-trip time |
| `select(.type == "message")` | Filter by response type |
| `select(.data.status >= 400)` | Filter by condition |
| `{a: .data.x, b: .data.y}` | Build custom objects |
| `.data \| keys` | Get object keys |
| `.data.items \| length` | Count array elements |
| `.data.body.items[] \| select(.active)` | Filter array items |
