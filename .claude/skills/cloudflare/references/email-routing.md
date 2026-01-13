# Cloudflare Email Routing Reference

## Overview

Cloudflare Email Routing allows you to create email addresses for your domain and forward them to your existing email accounts. Free for all Cloudflare plans.

## Setup Requirements

1. Domain must be on Cloudflare (DNS managed by Cloudflare)
2. Email Routing must be enabled for the zone
3. Destination email must be verified

## Enable Email Routing

First time setup must be done in dashboard:
1. Go to Cloudflare Dashboard > Your Domain > Email > Email Routing
2. Click "Get Started"
3. Cloudflare will add required MX and TXT records automatically

## Required DNS Records

### MX Records (for receiving email)

```bash
# MX Record 1
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route1.mx.cloudflare.net",
    "priority": 69,
    "ttl": 1
  }'

# MX Record 2
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route2.mx.cloudflare.net",
    "priority": 46,
    "ttl": 1
  }'

# MX Record 3
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route3.mx.cloudflare.net",
    "priority": 89,
    "ttl": 1
  }'
```

### SPF Record (for email authentication)

```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "TXT",
    "name": "@",
    "content": "v=spf1 include:_spf.mx.cloudflare.net ~all",
    "ttl": 1
  }'
```

### DMARC Record (optional but recommended)

```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "TXT",
    "name": "_dmarc",
    "content": "v=DMARC1; p=none; rua=mailto:dmarc@example.com",
    "ttl": 1
  }'
```

## Destination Addresses

### Add Destination Email (must verify)

```bash
curl -X POST "https://api.cloudflare.com/client/v4/accounts/{account_id}/email/routing/addresses" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "email": "your-real-email@gmail.com"
  }'
```

A verification email will be sent. User must click the link.

### List Destination Addresses

```bash
curl -X GET "https://api.cloudflare.com/client/v4/accounts/{account_id}/email/routing/addresses" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

### Delete Destination Address

```bash
curl -X DELETE "https://api.cloudflare.com/client/v4/accounts/{account_id}/email/routing/addresses/{address_id}" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## Routing Rules

### Create Specific Address Rule

Forward `contact@example.com` to `me@gmail.com`:

```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/email/routing/rules" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Forward contact",
    "enabled": true,
    "matchers": [
      {
        "type": "literal",
        "field": "to",
        "value": "contact@example.com"
      }
    ],
    "actions": [
      {
        "type": "forward",
        "value": ["me@gmail.com"]
      }
    ]
  }'
```

### Create Catch-All Rule

Forward all unmatched emails:

```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/email/routing/rules" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Catch-all",
    "enabled": true,
    "matchers": [
      {
        "type": "all"
      }
    ],
    "actions": [
      {
        "type": "forward",
        "value": ["me@gmail.com"]
      }
    ]
  }'
```

### Drop Unwanted Emails

```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/email/routing/rules" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Drop spam",
    "enabled": true,
    "matchers": [
      {
        "type": "literal",
        "field": "to",
        "value": "spam@example.com"
      }
    ],
    "actions": [
      {
        "type": "drop"
      }
    ]
  }'
```

### List Rules

```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones/{zone_id}/email/routing/rules" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

### Delete Rule

```bash
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/{zone_id}/email/routing/rules/{rule_id}" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## Action Types

| Type | Description | Value |
|------|-------------|-------|
| `forward` | Forward to destination | Array of emails |
| `drop` | Silently discard | None |
| `worker` | Process with Worker | Worker script name |

## Matcher Types

| Type | Field | Description |
|------|-------|-------------|
| `literal` | `to` | Exact email match |
| `all` | - | Catch-all (matches everything) |

## Common Patterns

### Multiple Addresses to Same Destination

Create separate rules for each address:

```bash
# contact@
# support@
# hello@
# All forward to same destination
```

### Department Routing

Different addresses to different destinations:

```bash
# sales@ -> sales-team@company.com
# support@ -> support-team@company.com
# billing@ -> finance@company.com
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Emails not arriving | MX records wrong | Verify 3 MX records exist |
| Destination not verified | Email not clicked | Check spam folder for verification |
| Rule not working | Priority conflict | Check rule order in dashboard |
| Emails going to spam | Missing SPF/DMARC | Add TXT records |

## Limits

- 200 routing rules per zone
- 200 destination addresses per account
- No sending capability (receive/forward only)
