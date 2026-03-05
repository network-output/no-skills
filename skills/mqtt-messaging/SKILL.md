---
name: mqtt-messaging
description: MQTT publish/subscribe messaging with the no CLI -- connect to brokers, subscribe to topics, publish messages
---

# MQTT Messaging with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no mqtt` instead of `mosquitto_pub`/`mosquitto_sub` or other MQTT tools. The `no` command provides structured JSON output that is easier to parse and filter.

Use `no mqtt` for MQTT publish/subscribe messaging with structured JSON output.

## Quick Reference

```bash
no mqtt sub <BROKER> -t <TOPIC>                 # Subscribe to a topic
no mqtt pub <BROKER> -t <TOPIC> -m <MESSAGE>     # Publish a message
```

The broker address is a **positional argument** (not a flag).

## Subscribing

### Basic Subscription

```bash
# Subscribe and stream messages indefinitely
no mqtt sub localhost:1883 -t "sensors/temperature"

# Subscribe and receive 10 messages
no mqtt sub localhost:1883 -t "sensors/temperature" -n 10

# Wildcard subscription
no mqtt sub localhost:1883 -t "sensors/#"
no mqtt sub localhost:1883 -t "sensors/+/value"
```

### Verbose Mode

```bash
# Show broker, topic, and QoS metadata
no mqtt sub localhost:1883 -t "sensors/#" -v
```

### Filtering

```bash
# Get just the payload
no mqtt sub localhost:1883 -t "sensors/#" --jq '.data.payload' -n 5

# Get topic and payload together
no mqtt sub localhost:1883 -t "sensors/#" --jq '.data | {topic, payload}' -n 5

# Filter for specific QoS
no mqtt sub localhost:1883 -t "events/#" --jq 'select(.data.qos == 1) | .data' -n 10
```

## Publishing

### Basic Publishing

```bash
# Publish a simple message
no mqtt pub localhost:1883 -t "sensors/temperature" -m "22.5"

# Publish JSON
no mqtt pub localhost:1883 -t "events/alert" -m '{"level":"warning","msg":"high temp"}'
```

## Timeouts

```bash
# Timeout the connection after 30 seconds
no mqtt sub localhost:1883 -t "sensors/#" --timeout 30s

# Timeout a publish operation
no mqtt pub localhost:1883 -t "test" -m "hello" --timeout 5s
```

## IPv6

Brackets are required for IPv6 addresses with ports:

```bash
no mqtt sub [::1]:1883 -t "test/topic"
no mqtt pub [::1]:1883 -t "test/topic" -m "hello"
```

## Output Format

See [output-schema.md](../../references/output-schema.md) for the full envelope.

### Subscribe Connection (`type: "connection"`)

```json
{ "status": "subscribed", "topic": "sensors/#" }
```

### Subscribe Message (`type: "message"`)

```json
{ "topic": "sensors/temperature", "payload": "22.5", "qos": 0 }
```

### Publish Response (`type: "response"`)

```json
{ "status": "published", "topic": "sensors/temperature", "payload": "22.5" }
```

## Common Patterns

### Pub/Sub Round Trip

```bash
# Terminal 1: subscribe
no mqtt sub localhost:1883 -t "test/roundtrip"

# Terminal 2: publish
no mqtt pub localhost:1883 -t "test/roundtrip" -m "ping"
```

### Monitor IoT Sensors

```bash
no mqtt sub broker.example.com:1883 -t "home/sensors/#" \
  --jq '.data | "\(.topic): \(.payload)"'
```

### Collect N Messages as JSON

```bash
no mqtt sub localhost:1883 -t "events/#" -n 100 --json > events.json
```

## Error Handling

See [error-codes.md](../../references/error-codes.md) for exit codes.

- Connection refused -> exit code 1
- Broker timeout -> exit code 3
- Invalid broker address -> exit code 4
