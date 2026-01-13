---
name: cloudflare
description: "Setup domains in Cloudflare with DNS for Clerk, Vercel, and email routing. Use when adding new domains, configuring DNS records, or setting up email redirects."
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, WebSearch
---

# Cloudflare Domain Setup

Automate the complete domain setup workflow: Cloudflare DNS, Clerk integration, Vercel deployment, and email routing.

## Prerequisites

### Authentication (Choose One)

**Option 1: API Token (Recommended)**
```bash
# Add to .env.local
CLOUDFLARE_API_TOKEN="your-api-token"
CLOUDFLARE_ACCOUNT_ID="your-account-id"
```

Create token at: https://dash.cloudflare.com/profile/api-tokens
Required permissions:
- Zone:DNS:Edit
- Zone:Zone:Read
- Email Routing Addresses:Edit
- Email Routing Rules:Edit

**Option 2: Wrangler CLI**
```bash
# Install wrangler
bun add -g wrangler

# Login (opens browser)
wrangler login

# Verify
wrangler whoami
```

### Other Tools
```bash
# Vercel CLI (required)
bun add -g vercel
vercel login
```

## Workflow

When setting up a new domain, follow these steps:

### Step 1: Gather Information

Ask the user for:
1. **Domain name** (e.g., `example.com`)
2. **Clerk DNS records** (paste from Clerk dashboard)
3. **Vercel project name** (e.g., `my-app`)
4. **Email addresses** to create (e.g., `contact`, `support`)
5. **Redirect target email** (e.g., `me@gmail.com`)

### Step 2: Get Zone ID

```bash
# If using API token
curl -X GET "https://api.cloudflare.com/client/v4/zones?name=DOMAIN" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" | jq '.result[0].id'

# If using wrangler
wrangler pages project list  # Shows associated zones
```

### Step 3: Create DNS Records for Clerk

Clerk provides specific DNS records for each project. Common patterns:

```bash
# Example: CNAME record
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "CNAME",
    "name": "clerk",
    "content": "frontend-api.clerk.dev",
    "ttl": 1,
    "proxied": false
  }'

# Example: TXT record for verification
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "TXT",
    "name": "@",
    "content": "clerk-verification=xxxxx",
    "ttl": 1
  }'
```

### Step 4: Add Domain to Vercel

```bash
# Add domain to Vercel project
vercel domains add DOMAIN --scope=TEAM_SLUG

# Or link to specific project
vercel domains add DOMAIN PROJECT_NAME
```

Then create Vercel DNS records:

```bash
# A record for root domain
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "A",
    "name": "@",
    "content": "76.76.21.21",
    "ttl": 1,
    "proxied": false
  }'

# CNAME for www subdomain
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "CNAME",
    "name": "www",
    "content": "cname.vercel-dns.com",
    "ttl": 1,
    "proxied": false
  }'
```

### Step 5: Setup Email Routing

First, enable email routing for the zone (do this in Cloudflare dashboard first time).

Then create routing rules:

```bash
# Create destination address (must be verified first)
curl -X POST "https://api.cloudflare.com/client/v4/accounts/ACCOUNT_ID/email/routing/addresses" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "email": "your-main-email@gmail.com"
  }'

# Create routing rule for contact@domain.com
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/email/routing/rules" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "name": "Forward contact",
    "enabled": true,
    "matchers": [{"type": "literal", "field": "to", "value": "contact@DOMAIN"}],
    "actions": [{"type": "forward", "value": ["your-main-email@gmail.com"]}]
  }'
```

Required MX records for email routing:
```bash
# MX records for Cloudflare Email Routing
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route1.mx.cloudflare.net",
    "priority": 69,
    "ttl": 1
  }'

curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route2.mx.cloudflare.net",
    "priority": 46,
    "ttl": 1
  }'

curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "MX",
    "name": "@",
    "content": "route3.mx.cloudflare.net",
    "priority": 89,
    "ttl": 1
  }'

# TXT record for SPF
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "TXT",
    "name": "@",
    "content": "v=spf1 include:_spf.mx.cloudflare.net ~all",
    "ttl": 1
  }'
```

### Step 6: Verification Checklist

After setup, verify:

```bash
# List all DNS records
curl -X GET "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" | jq '.result[] | {type, name, content}'

# Check Vercel domain status
vercel domains inspect DOMAIN

# Test email routing (send test email to contact@DOMAIN)
```

## Interactive Prompts Template

When running `/cloudflare`, ask:

```
What domain are you setting up?
> example.com

Paste the Clerk DNS records from your Clerk dashboard:
> [user pastes records]

What's the Vercel project name?
> my-saas-app

What email addresses should I create? (comma-separated)
> contact, support, hello

What email should these redirect to?
> myemail@gmail.com
```

## Common DNS Record Types

| Type | Use Case | Proxied |
|------|----------|---------|
| A | Root domain to IP | No (for Vercel) |
| CNAME | Subdomain to hostname | No (for Clerk/Vercel) |
| TXT | Verification, SPF | N/A |
| MX | Email routing | N/A |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Zone not found | Domain must be added to Cloudflare first |
| DNS propagation slow | Wait 5-10 minutes, check with `dig` |
| Email not forwarding | Verify destination email first |
| Vercel 404 | Check DNS proxied=false for Vercel records |
| Clerk verification failed | Ensure TXT record is on root (@) |

## Useful Commands

```bash
# Check DNS propagation
dig DOMAIN +short
dig DOMAIN MX +short
dig DOMAIN TXT +short

# List zones in account
curl -X GET "https://api.cloudflare.com/client/v4/zones" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" | jq '.result[] | {name, id}'

# Delete a DNS record
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records/RECORD_ID" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```
