# Cloudflare R2 Storage Reference

## Overview

R2 is Cloudflare's S3-compatible object storage with zero egress fees. Use for user uploads, static assets, backups, and any file storage needs.

## Prerequisites

```bash
# Wrangler CLI (recommended)
bun add -g wrangler
wrangler login

# Or API Token with permissions:
# - Account:R2:Edit
# - Account:R2:Read
```

## Wrangler Commands

### Bucket Management

```bash
# Create bucket
wrangler r2 bucket create my-bucket

# List all buckets
wrangler r2 bucket list

# Get bucket info
wrangler r2 bucket info my-bucket

# Delete bucket (must be empty)
wrangler r2 bucket delete my-bucket
```

### Object Management

```bash
# List objects in bucket
wrangler r2 object list my-bucket

# Upload file
wrangler r2 object put my-bucket/path/file.png --file=./local-file.png

# Download file
wrangler r2 object get my-bucket/path/file.png --file=./downloaded.png

# Delete object
wrangler r2 object delete my-bucket/path/file.png
```

## API Operations

### Create Bucket

```bash
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/{account_id}/r2/buckets" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"name": "my-bucket", "locationHint": "wnam"}'
```

Location hints: `wnam` (Western North America), `enam` (Eastern NA), `weur` (Western Europe), `eeur` (Eastern Europe), `apac` (Asia Pacific)

### List Buckets

```bash
curl -X GET "https://api.cloudflare.com/client/v4/accounts/{account_id}/r2/buckets" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

### Delete Bucket

```bash
curl -X DELETE "https://api.cloudflare.com/client/v4/accounts/{account_id}/r2/buckets/{bucket_name}" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
```

## CORS Configuration

### For User Uploads (Images, Files)

```bash
# Create cors.json
cat > cors.json << 'EOF'
{
  "corsRules": [
    {
      "allowedOrigins": ["https://myapp.com", "http://localhost:3000"],
      "allowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
      "allowedHeaders": ["*"],
      "exposeHeaders": ["ETag", "Content-Length"],
      "maxAgeSeconds": 3600
    }
  ]
}
EOF

# Apply CORS via wrangler
wrangler r2 bucket cors put my-bucket --file=cors.json

# View current CORS
wrangler r2 bucket cors get my-bucket

# Remove CORS
wrangler r2 bucket cors delete my-bucket
```

### Common CORS Presets

**Public read (CDN/static assets):**
```json
{
  "corsRules": [
    {
      "allowedOrigins": ["*"],
      "allowedMethods": ["GET", "HEAD"],
      "allowedHeaders": ["*"],
      "maxAgeSeconds": 86400
    }
  ]
}
```

**Restricted upload (specific domains):**
```json
{
  "corsRules": [
    {
      "allowedOrigins": ["https://myapp.com"],
      "allowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "allowedHeaders": ["Content-Type", "Authorization"],
      "maxAgeSeconds": 3600
    }
  ]
}
```

## Public Access

### Enable R2.dev Subdomain

Via Cloudflare Dashboard:
1. Go to R2 > Bucket > Settings
2. Enable "Public access"
3. Get URL: `https://pub-{hash}.r2.dev`

### Custom Domain Setup

1. **Add CNAME record** in Cloudflare DNS:
```bash
# For bucket at uploads.myapp.com
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "CNAME",
    "name": "uploads",
    "content": "{account_id}.r2.cloudflarestorage.com",
    "ttl": 1,
    "proxied": true
  }'
```

2. **Enable custom domain** in R2 settings (via dashboard or API)

3. **Verify** the domain is active:
```bash
curl -I https://uploads.myapp.com/test-file.png
```

## S3-Compatible API

R2 is S3-compatible. To use with AWS SDK or S3 tools:

### Get API Credentials

1. Go to R2 > Manage R2 API Tokens
2. Create token with Object Read & Write permissions
3. Note the Access Key ID and Secret Access Key

### Environment Variables

```bash
# For S3-compatible tools
R2_ACCESS_KEY_ID="your-access-key"
R2_SECRET_ACCESS_KEY="your-secret-key"
R2_ENDPOINT="https://{account_id}.r2.cloudflarestorage.com"
```

### Using with AWS SDK (Node.js)

```typescript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const r2 = new S3Client({
  region: "auto",
  endpoint: process.env.R2_ENDPOINT,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});

// Upload file
await r2.send(new PutObjectCommand({
  Bucket: "my-bucket",
  Key: "uploads/image.png",
  Body: fileBuffer,
  ContentType: "image/png",
}));
```

### Using with presigned URLs

```typescript
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";

// Generate upload URL (valid for 1 hour)
const uploadUrl = await getSignedUrl(r2, new PutObjectCommand({
  Bucket: "my-bucket",
  Key: "uploads/user-file.png",
}), { expiresIn: 3600 });

// Generate download URL
const downloadUrl = await getSignedUrl(r2, new GetObjectCommand({
  Bucket: "my-bucket",
  Key: "uploads/user-file.png",
}), { expiresIn: 3600 });
```

## Common Use Cases

### 1. User Uploads (Images, Documents)

```typescript
// Next.js API route for presigned upload URL
// app/api/upload/route.ts
export async function POST(request: Request) {
  const { filename, contentType } = await request.json();

  const key = `uploads/${Date.now()}-${filename}`;
  const uploadUrl = await getSignedUrl(r2, new PutObjectCommand({
    Bucket: "user-uploads",
    Key: key,
    ContentType: contentType,
  }), { expiresIn: 3600 });

  return Response.json({ uploadUrl, key });
}
```

### 2. Static Assets (Public)

- Enable public access via R2.dev or custom domain
- Use for images, CSS, JS served via CDN
- Set long cache headers

### 3. Backups (Private)

- Keep bucket private (no public access)
- Use presigned URLs for temporary access
- Consider lifecycle rules for old backups

## Lifecycle Rules

Automatically delete or transition objects:

```bash
# Create lifecycle.json
cat > lifecycle.json << 'EOF'
{
  "rules": [
    {
      "id": "delete-old-temp-files",
      "enabled": true,
      "conditions": {
        "prefix": "temp/"
      },
      "deleteObjectsAfterDays": 7
    },
    {
      "id": "delete-old-logs",
      "enabled": true,
      "conditions": {
        "prefix": "logs/"
      },
      "deleteObjectsAfterDays": 30
    }
  ]
}
EOF

# Apply lifecycle rules
wrangler r2 bucket lifecycle put my-bucket --file=lifecycle.json
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| CORS error | Check allowedOrigins includes your domain |
| 403 Forbidden | Verify API token has R2 permissions |
| Bucket not found | Check account ID and bucket name |
| Upload fails | Verify Content-Type header matches file |
| Custom domain 404 | Ensure CNAME is proxied (orange cloud) |
| Slow uploads | Use presigned URLs for direct browser upload |

## Pricing

- Storage: $0.015/GB per month
- Class A operations (PUT, POST, LIST): $4.50 per million
- Class B operations (GET, HEAD): $0.36 per million
- **Egress: FREE** (no bandwidth charges)

## Useful Commands

```bash
# Check bucket size
wrangler r2 bucket info my-bucket

# Upload entire directory
for f in ./assets/*; do
  wrangler r2 object put my-bucket/assets/$(basename $f) --file=$f
done

# Generate presigned URL (via AWS CLI)
aws s3 presign s3://my-bucket/file.png \
  --endpoint-url $R2_ENDPOINT \
  --expires-in 3600
```
