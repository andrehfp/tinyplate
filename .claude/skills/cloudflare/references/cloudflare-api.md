# Cloudflare API Reference

## Authentication

### API Token (Recommended)
```bash
# Header format
-H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

Create at: https://dash.cloudflare.com/profile/api-tokens

Required permissions:
- **Zone:DNS:Edit** - Create/modify DNS records
- **Zone:Zone:Read** - List and read zone info
- **Account:Email Routing Addresses:Edit** - Manage destination emails
- **Zone:Email Routing Rules:Edit** - Create routing rules

### Global API Key (Legacy)
```bash
-H "X-Auth-Email: your@email.com"
-H "X-Auth-Key: your-global-api-key"
```

## Base URL
```
https://api.cloudflare.com/client/v4
```

## Zone Operations

### List Zones
```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json"
```

### Get Zone by Domain
```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones?name=example.com" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

Response:
```json
{
  "result": [{
    "id": "zone-id-here",
    "name": "example.com",
    "status": "active"
  }]
}
```

## DNS Record Operations

### List DNS Records
```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

### Create DNS Record
```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "A",
    "name": "@",
    "content": "192.0.2.1",
    "ttl": 1,
    "proxied": false
  }'
```

### Record Types

| Type | Fields | Example Content |
|------|--------|-----------------|
| A | name, content, ttl, proxied | `192.0.2.1` |
| AAAA | name, content, ttl, proxied | `2001:db8::1` |
| CNAME | name, content, ttl, proxied | `target.example.com` |
| TXT | name, content, ttl | `v=spf1 include:...` |
| MX | name, content, priority, ttl | `mail.example.com` |

### Special Name Values
- `@` - Root domain
- `*` - Wildcard
- `subdomain` - Specific subdomain

### TTL Values
- `1` - Automatic (recommended)
- `60` - 1 minute
- `3600` - 1 hour
- `86400` - 1 day

### Update DNS Record
```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records/{record_id}" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "content": "new-value"
  }'
```

### Delete DNS Record
```bash
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records/{record_id}" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## Error Handling

### Response Format
```json
{
  "success": true,
  "errors": [],
  "messages": [],
  "result": { ... }
}
```

### Common Errors

| Code | Message | Solution |
|------|---------|----------|
| 9109 | Invalid access token | Check token permissions |
| 81057 | Record already exists | Delete or update existing |
| 1004 | DNS validation error | Check record format |

## Rate Limits

- 1200 requests per 5 minutes per user
- Applies across all API endpoints
- Returns `429 Too Many Requests` when exceeded
