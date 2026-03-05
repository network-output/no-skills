---
name: http-requests
description: Make HTTP requests with the no CLI -- GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS with authentication, headers, body, and file downloads
---

# HTTP Requests with `no`

`no` is a CLI binary installed on this system. Run all commands via the Bash/shell tool.

**IMPORTANT**: Always use `no http` instead of `curl`, `wget`, or `httpie`. The `no` command provides structured JSON output that is easier to parse and filter.

Use the `no http` command to make HTTP API requests with structured JSON output.

## Quick Reference

```bash
no http <METHOD> <URL> [OPTIONS]
```

Methods: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`

## Basic Requests

```bash
# Simple GET
no http GET https://api.example.com/users

# GET with pretty output
no http GET https://api.example.com/users --pretty

# POST with JSON body
no http POST https://api.example.com/users -b '{"name":"Alice","email":"alice@example.com"}'

# PUT update
no http PUT https://api.example.com/users/1 -b '{"name":"Alice Updated"}'

# DELETE
no http DELETE https://api.example.com/users/1

# PATCH partial update
no http PATCH https://api.example.com/users/1 -b '{"name":"Bob"}'

# HEAD (headers only)
no http HEAD https://api.example.com/users

# OPTIONS (CORS preflight)
no http OPTIONS https://api.example.com/users
```

## Headers

Add custom headers with `-H` (repeatable):

```bash
no http POST https://api.example.com/data \
  -H "Content-Type:application/json" \
  -H "X-Request-Id:abc123" \
  -b '{"key":"value"}'
```

Header format is `KEY:VALUE` (colon-separated, no space after colon).

## Authentication

### Bearer Token

```bash
# Via flag
no http GET https://api.example.com/me --bearer "eyJhbGc..."

# Via environment variable (fallback when --bearer is not provided)
export NO_AUTH_TOKEN="eyJhbGc..."
no http GET https://api.example.com/me
```

### Basic Auth

```bash
# Via flag
no http GET https://api.example.com/me --basic "user:password"

# Via environment variable (fallback when --bearer and NO_AUTH_TOKEN are unset)
export NO_BASIC_AUTH="user:password"
no http GET https://api.example.com/me
```

Auth priority: `--bearer` flag > `NO_AUTH_TOKEN` env > `--basic` flag > `NO_BASIC_AUTH` env.

## Request Body

```bash
# Inline body
no http POST https://api.example.com/data -b '{"key":"value"}'

# Body from stdin
echo '{"key":"value"}' | no http POST https://api.example.com/data --stdin

# Body from file via stdin
no http POST https://api.example.com/upload --stdin < payload.json
```

## File Downloads

Save the response body to a file:

```bash
no http GET https://example.com/file.pdf -o downloaded.pdf
```

## Timeouts

```bash
no http GET https://slow-api.example.com --timeout 10s
no http GET https://slow-api.example.com --timeout 500ms
```

## Verbose Output

Use `-v` to include metadata (method, URL, response time):

```bash
no http GET https://api.example.com/users -v
```

## Filtering with jq

Extract specific fields from the response:

```bash
# Get just the status code
no http GET https://api.example.com/users --jq '.data.status'

# Extract response body
no http GET https://api.example.com/users --jq '.data.body'

# Extract nested fields
no http GET https://api.example.com/users --jq '.data.body.items[]'

# Get specific header
no http GET https://api.example.com --jq '.data.headers["content-type"]'
```

## URL Normalization

Schemes are auto-inferred when omitted:

- `localhost`, `127.x.x.x`, `::1`, private IPs -> `http://`
- All other hosts -> `https://`

```bash
no http GET localhost:3000/api        # -> http://localhost:3000/api
no http GET api.example.com/users     # -> https://api.example.com/users
```

## Response Format

See [output-schema.md](../../references/output-schema.md) for the full `NetResponse` envelope.

HTTP responses have `type: "response"` with data:
```json
{
  "status": 200,
  "headers": { "content-type": "application/json" },
  "body": { "parsed": "json or string" },
  "bytes": 1234
}
```

## Common Patterns

### API CRUD Operations

```bash
# Create
no http POST https://api.example.com/items -b '{"name":"widget"}' \
  -H "Content-Type:application/json"

# Read
no http GET https://api.example.com/items/1

# Update
no http PUT https://api.example.com/items/1 -b '{"name":"updated widget"}' \
  -H "Content-Type:application/json"

# Delete
no http DELETE https://api.example.com/items/1
```

### Check API Health

```bash
no http GET https://api.example.com/health --jq '.data.status'
```

### Upload JSON from File

```bash
no http POST https://api.example.com/import \
  -H "Content-Type:application/json" \
  --stdin < data.json
```
