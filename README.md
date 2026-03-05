# no -- AI Skills Plugin

AI skills for the `no` (network-output) CLI tool. These skills teach AI agents how to use `no` for networking tasks across HTTP, WebSocket, TCP, UDP, MQTT, SSE, DNS, Ping, and WHOIS.

Compatible with any agent that supports the [Agent Skills](https://agentskills.io) open standard: Claude Code, Codex CLI, Cursor, Gemini CLI, and others.

## Skills

| Skill | Description |
|---|---|
| `http-requests` | HTTP API testing (GET, POST, PUT, DELETE, auth, headers, body, downloads) |
| `websocket-debugging` | WebSocket connection debugging (listen, send, lifecycle monitoring) |
| `network-diagnostics` | DNS lookups, ICMP ping, WHOIS queries |
| `mqtt-messaging` | MQTT publish/subscribe messaging |
| `tcp-udp-testing` | Raw TCP connect/listen, UDP send/listen |
| `sse-monitoring` | Server-Sent Events stream monitoring |
| `output-filtering` | jq filtering (`--jq` flag and standalone `no jq`) |

## Installation

### Prerequisites

Install `no` first:

```bash
# macOS
brew install network-output/tap/no

# Cargo
cargo install network-output

# See https://network-output.com for all installation options
```

### Using the Plugin

#### From GitHub

```bash
claude --plugin network-output/no-skills
```

#### Built into `no`

```bash
no skills install
```

This writes the plugin to `~/.claude/plugins/no/`.

#### Local (from this repo)

```bash
claude --plugin-dir ./claude-plugin
```

## Structure

```
claude-plugin/
  .claude-plugin/
    plugin.json              # Plugin manifest
  skills/
    http-requests/SKILL.md
    websocket-debugging/SKILL.md
    network-diagnostics/SKILL.md
    mqtt-messaging/SKILL.md
    tcp-udp-testing/SKILL.md
    sse-monitoring/SKILL.md
    output-filtering/SKILL.md
  references/
    cli-reference.md         # Complete CLI syntax
    output-schema.md         # NetResponse envelope and per-protocol shapes
    error-codes.md           # Exit codes and ErrorCode variants
```

## Development

### Validate Plugin Structure

```bash
just validate-plugin
```

### Test Locally

```bash
claude --plugin-dir ./claude-plugin
```

Then try:
- Ask "make an HTTP request to httpbin.org/get"
- Ask "ping google.com 3 times"
- Ask "look up DNS records for example.com"

## License

Apache-2.0 -- see [LICENSE](./LICENSE).
