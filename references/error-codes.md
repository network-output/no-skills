# Error Codes

All errors are emitted as a `NetResponse` with `type: "error"` and a structured `data` payload containing `code` and `message`.

## ErrorCode Variants

| Code | Serialized Value | Exit Code | When It Occurs |
|---|---|---|---|
| `ConnectionRefused` | `CONNECTION_REFUSED` | 1 | TCP connection actively refused by remote host |
| `DnsResolution` | `DNS_RESOLUTION` | 1 | Hostname could not be resolved to an IP address |
| `IoError` | `IO_ERROR` | 1 | Generic I/O failure (read/write errors, bind failures) |
| `ProtocolError` | `PROTOCOL_ERROR` | 2 | Protocol-level failure (unexpected frames, malformed responses) |
| `TlsError` | `TLS_ERROR` | 2 | TLS handshake or certificate verification failed |
| `ConnectionTimeout` | `CONNECTION_TIMEOUT` | 3 | Connection or read operation exceeded timeout |
| `InvalidInput` | `INVALID_INPUT` | 4 | Bad user input (invalid URL, header format, method, port, jq expression) |

## Exit Code Summary

| Exit Code | Category | Error Codes |
|---|---|---|
| 0 | Success | (no error) |
| 1 | Connection / DNS / I/O | `CONNECTION_REFUSED`, `DNS_RESOLUTION`, `IO_ERROR` |
| 2 | Protocol / TLS | `PROTOCOL_ERROR`, `TLS_ERROR` |
| 3 | Timeout | `CONNECTION_TIMEOUT` |
| 4 | Invalid input | `INVALID_INPUT` |

## Error Response Shape

```json
{
  "type": "error",
  "protocol": "http",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "data": {
    "code": "CONNECTION_REFUSED",
    "message": "connection refused"
  }
}
```

## Handling Errors Programmatically

Check the exit code to determine the error category:

```bash
no http GET https://unreachable.example.com
echo $?  # 1 (connection error)

no http GET https://expired-cert.example.com
echo $?  # 2 (TLS error)

no http GET https://slow.example.com --timeout 1s
echo $?  # 3 (timeout)

no http INVALID https://example.com
echo $?  # 4 (invalid input)
```

For JSON consumers, parse the `data.code` field:

```bash
no http GET https://unreachable.example.com --json 2>/dev/null | no jq '.data.code'
# CONNECTION_REFUSED
```
