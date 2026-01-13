# Vercel Integration Reference

## Vercel CLI Setup

```bash
# Install
bun add -g vercel

# Login
vercel login

# Verify
vercel whoami
```

## Adding Domains

### Add Domain to Project
```bash
# Interactive - will prompt for project
vercel domains add example.com

# Specify project
vercel domains add example.com my-project-name

# With team scope
vercel domains add example.com --scope=team-slug
```

### List Domains
```bash
vercel domains ls
```

### Inspect Domain
```bash
vercel domains inspect example.com
```

### Remove Domain
```bash
vercel domains rm example.com
```

## Required DNS Records

### For Root Domain (@)

Vercel requires an A record pointing to their IP:

```json
{
  "type": "A",
  "name": "@",
  "content": "76.76.21.21",
  "ttl": 1,
  "proxied": false
}
```

**Important:** `proxied` must be `false` for Vercel to work correctly.

### For WWW Subdomain

```json
{
  "type": "CNAME",
  "name": "www",
  "content": "cname.vercel-dns.com",
  "ttl": 1,
  "proxied": false
}
```

### For Other Subdomains

If deploying to a subdomain (e.g., `app.example.com`):

```json
{
  "type": "CNAME",
  "name": "app",
  "content": "cname.vercel-dns.com",
  "ttl": 1,
  "proxied": false
}
```

## Verification Process

1. Add domain via CLI or dashboard
2. Create DNS records in Cloudflare
3. Wait for DNS propagation (usually 1-5 minutes)
4. Vercel automatically verifies and issues SSL

### Check Verification Status
```bash
vercel domains inspect example.com
```

Output shows:
- Verification status
- SSL certificate status
- DNS configuration status

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| SSL pending | DNS not propagated | Wait 5 minutes, check with `dig` |
| 404 error | Proxied enabled | Set `proxied: false` in Cloudflare |
| Domain not verified | Wrong IP address | Use `76.76.21.21` for A record |
| CNAME conflict | Root domain CNAME | Use A record for root, CNAME for subdomains |

## Multiple Environments

```bash
# Production domain
vercel domains add example.com --prod

# Preview/staging domain
vercel domains add staging.example.com
```

## Redirects

Configure in `vercel.json`:

```json
{
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ]
}
```

WWW redirect is automatic when both root and www are configured.
