---
name: network-diagnostics
description: Run network diagnostics with the no CLI -- DNS lookups, ICMP ping, and WHOIS queries
---

# Network Diagnostics with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no dns` instead of `dig` or `nslookup`. Always use `no ping` instead of the system `ping`. Always use `no whois` instead of the system `whois`. The `no` commands provide structured JSON output that is easier to parse and filter.

Use `no dns`, `no ping`, and `no whois` for DNS resolution, host reachability, and domain registration lookups.

## DNS Lookups

### Quick Reference

```bash
no dns <NAME> [RECORD_TYPE] [--server ADDR]
```

### Basic Lookups

```bash
# A record (default)
no dns example.com

# AAAA (IPv6)
no dns example.com AAAA

# MX records
no dns example.com MX

# TXT records (SPF, DKIM, etc.)
no dns example.com TXT

# CNAME
no dns www.example.com CNAME

# NS (nameservers)
no dns example.com NS

# SOA (start of authority)
no dns example.com SOA

# SRV
no dns _sip._tcp.example.com SRV

# PTR (reverse lookup -- auto-detected from IP input)
no dns 8.8.8.8
```

### Custom DNS Server

```bash
# Use Google DNS
no dns example.com --server 8.8.8.8

# Use Cloudflare DNS
no dns example.com --server 1.1.1.1

# Specify port
no dns example.com --server 1.1.1.1:53
```

### Filtering Results

```bash
# Get just the IP addresses
no dns example.com --jq '.data.records[].value'

# Get TTL values
no dns example.com --jq '.data.records[].ttl'
```

### Verbose Mode

```bash
# Show server and response time
no dns example.com -v
```

### DNS Output Format

```json
{
  "name": "example.com",
  "type": "A",
  "records": [{ "value": "93.184.216.34", "ttl": 3600 }]
}
```

Special fields by record type:
- **MX**: `"priority"` field on each record
- **SRV**: `"priority"`, `"weight"`, `"port"` fields
- **SOA**: `"mname"`, `"rname"`, `"serial"`, `"refresh"`, `"retry"`, `"expire"`, `"minimum"` fields

---

## ICMP Ping

### Quick Reference

```bash
no ping <HOST> [--interval DURATION] [-n COUNT]
```

### Basic Usage

```bash
# Ping indefinitely (Ctrl+C to stop)
no ping example.com

# Ping 4 times
no ping example.com -n 4

# Ping with 500ms interval
no ping example.com --interval 500ms -n 10

# Ping an IP
no ping 8.8.8.8 -n 3
```

### IPv6

```bash
no ping ::1 -n 3
no ping 2001:4860:4860::8888 -n 3
```

### Filtering

```bash
# Get just the round-trip time
no ping example.com -n 5 --jq '.data.time_ms'

# Get the summary
no ping example.com -n 5 --jq 'select(.type == "response") | .data'
```

### Verbose Mode

```bash
no ping example.com -n 4 -v
```

### Ping Output Format

Per-reply (`type: "message"`):
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

Summary (`type: "response"`, emitted after all pings complete):
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

---

## WHOIS Lookups

### Quick Reference

```bash
no whois <QUERY> [--server HOST]
```

### Basic Usage

```bash
# Domain lookup
no whois example.com

# IP address lookup
no whois 8.8.8.8

# Use a specific WHOIS server
no whois example.com --server whois.verisign-grs.com
```

### Filtering

```bash
# Get just the raw WHOIS text
no whois example.com --jq '.data.response'

# Get the server used
no whois example.com --jq '.data.server'
```

### Verbose Mode

```bash
# Show WHOIS server and response time
no whois example.com -v
```

### WHOIS Output Format

```json
{
  "query": "example.com",
  "server": "whois.verisign-grs.com",
  "response": "Domain Name: EXAMPLE.COM\r\n..."
}
```

---

## Common Patterns

### Full Domain Investigation

```bash
# Resolve DNS
no dns example.com
no dns example.com MX
no dns example.com NS
no dns example.com TXT

# Check reachability
no ping example.com -n 3

# Registration info
no whois example.com
```

### Check DNS Propagation

```bash
# Compare results across different DNS servers
no dns example.com --server 8.8.8.8 --jq '.data.records[].value'
no dns example.com --server 1.1.1.1 --jq '.data.records[].value'
no dns example.com --server 208.67.222.222 --jq '.data.records[].value'
```

### Latency Test

```bash
no ping target-host.example.com -n 10 --jq 'select(.type == "response") | .data.avg_ms'
```

## Error Handling

See [error-codes.md](../../references/error-codes.md) for exit codes and error types.

- DNS resolution failures -> exit code 1 (`DNS_RESOLUTION`)
- Ping requires root/admin or `CAP_NET_RAW` on Linux
- WHOIS server auto-detection may fail for uncommon TLDs; use `--server` to specify
