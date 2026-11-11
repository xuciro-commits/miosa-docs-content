---
slug: "data/storage"
title: "Storage / Buckets"
description: "Per-project object storage with signed-URL uploads and downloads."
section: "Databases & storage"
order: 4006
---

> **Tip: Available**
>
> Object storage is shipped. Buckets, objects, presigned URL (upload and download), quota enforcement, and per-tenant isolation are all live. Large direct uploads (&gt;5 MB) should use the presigned path; see the limits table below.

## What you get

Per project: an S3-compatible bucket, accessed through MIOSA endpoints. Credentials injected as env vars:
```bash

STORAGE_BUCKET=miosa-proj_xxx-uploads
STORAGE_ENDPOINT=https://storage.miosa.app
STORAGE_KEY_ID=...
STORAGE_KEY_SECRET       # from the bucket settings
```

Your code uses the standard S3 client libraries:
```typescript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3"

const s3 = new S3Client({
  endpoint: process.env.STORAGE_ENDPOINT,
  credentials: {
    accessKeyId: process.env.STORAGE_KEY_ID!,
    secretAccessKey: process.env.STORAGE_KEY_SECRET!,
  },
  region: "us-east-1",
})

await s3.send(new PutObjectCommand({
  Bucket: process.env.STORAGE_BUCKET,
  Key: "uploads/foo.png",
  Body: pngBuffer,
}))
```

## Use it your way: API, CLI, or SDK

No SDK required. Same bucket, three ways.

> Storage calls need an API key with the `storage:read` / `storage:write` scope. Mint one with `POST /api/v1/api-keys {"scopes": ["storage:read","storage:write"]}`. A plain key gets a 403 telling you which scope is missing.

**Raw API (curl):**
```bash
# Create a bucket
curl -X POST https://api.miosa.ai/api/v1/storage/buckets \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"name": "my-bucket"}'

# Upload an object (binary → application/octet-stream)
curl -X PUT "https://api.miosa.ai/api/v1/storage/buckets/BUCKET_ID/objects/path/file.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/octet-stream" --data-binary @file.txt

# Download it
curl "https://api.miosa.ai/api/v1/storage/buckets/BUCKET_ID/objects/path/file.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Presign a URL so a browser can upload/download directly (no proxy)
curl -X POST "https://api.miosa.ai/api/v1/storage/buckets/BUCKET_ID/presign" \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"key": "path/file.txt", "method": "GET"}'
```

**CLI:**
```bash
miosa storage create my-bucket
miosa storage list
miosa storage objects <bucket-id>
```

Upload, download and presigning have no dedicated subcommand; drive them with `miosa api`:

```bash
# Upload an object
miosa api PUT /storage/buckets/<bucket-id>/objects/path/file.txt \
  -H "Content-Type: application/octet-stream" -d @./file.txt

# Download it
miosa api GET /storage/buckets/<bucket-id>/objects/path/file.txt

# Presign a URL so a browser can upload/download directly (no proxy)
miosa api POST /storage/buckets/<bucket-id>/presign \
  -d '{"key": "path/file.txt", "method": "GET"}'
```

## Signed URLs for browser uploads

Don't proxy file uploads through your runtime. Mint a signed URL server-side; the browser uploads directly to storage:
```typescript
// Server
const url = await miosa.storage.signedUrl({
  bucket: process.env.STORAGE_BUCKET!,
  key: `users/${userId}/avatar.png`,
  operation: "put",
  expiresInSec: 300,
  contentType: "image/png",
})
return { uploadUrl: url }
```
```typescript
// Browser
await fetch(uploadUrl, {
  method: "PUT",
  body: file,
  headers: { "Content-Type": "image/png" },
})
```

Same pattern for downloads (`operation: "get"`). Signed URLs are scoped (single key, single operation, short expiry)  -  no risk of leaking bucket-wide credentials to a browser.

## Public vs private

Each object can be private (default; access only via signed URL) or public (served from a `*.storage.miosa.app` URL). Public objects can also be fronted by a custom domain via your project's [Domain](/docs/docker-host/domains) settings.

## Bandwidth and storage costs

Two billable dimensions:

- **Storage**  -  GB-month for stored bytes.
- **Egress**  -  GB out from MIOSA to the public internet. Storage-to-runtime within MIOSA is free.

[Attribution](/docs/platform/attribution) is carried on every event for chargeback.

## Limits

| Limit | Default | Tunable |
|---|---|---|
| Storage per project | 1 GB | Yes (plan-dependent) |
| Direct API upload size | 5 MB verified | Larger uploads should use presigned/direct storage |
| Max object size | 5 GB target | Up to 100 GB target for multipart |
| List keys per call | 1000 | Standard S3 pagination |

> **Note**
>
> The latest live storage smoke on June 9, 2026 verified create, upload,
> download, SHA-256 validation, and cleanup through `10 / 10` sequential `5 MB`
> objects. `8 MB` and `10 MB` direct raw uploads currently fail before storage, so
> large files should use presigned URLs or S3-compatible direct upload flows until
> the streaming API path is promoted. See [Benchmarks](/docs/benchmarks) for the
> measured table and the ComputeSDK storage comparison.

## Sandboxes can use the bucket too

The same env vars are injected into sandboxes for dev / testing. Upload from inside a sandbox:
```typescript
await sandbox.exec({
  cmd: "aws",
  args: ["s3", "cp", "/workspace/build.tar.gz", `s3://${process.env.STORAGE_BUCKET}/builds/`],
  env: {
    AWS_ACCESS_KEY_ID: process.env.STORAGE_KEY_ID,
    AWS_SECRET_ACCESS_KEY: process.env.STORAGE_KEY_SECRET,
    AWS_ENDPOINT_URL: process.env.STORAGE_ENDPOINT,
  },
})
```

## Bring your own bucket?

Yes. Set the env vars to point at an external S3-compatible bucket (real AWS, Backblaze B2, R2, etc.). MIOSA injects whatever you set.

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Create a bucket
bucket = miosa.storage.create_bucket("my-uploads", public=False)
bucket_id = bucket["id"]

# Upload bytes
with open("report.pdf", "rb") as f:
    miosa.storage.put_object(bucket_id, "reports/report.pdf", f.read(),
                             content_type="application/pdf")

# Mint a presigned URL for browser upload (PUT)
presigned = miosa.storage.presign(
    bucket_id,
    key="avatars/user-123.png",
    operation="put",
    expires_in_sec=300,
    content_type="image/png",
)
print(presigned["url"])

# Mint a presigned URL for browser download (GET)
download = miosa.storage.presign(
    bucket_id,
    key="reports/report.pdf",
    operation="get",
    expires_in_sec=3600,
)
print(download["url"])

# List objects with a prefix
objects = miosa.storage.list_objects(bucket_id, prefix="reports/")
for obj in objects:
    print(obj["key"], obj["size"])

# Delete a bucket
miosa.storage.delete_bucket(bucket_id)
```

## See also

- [Overview](/docs/data/overview)  -  the broader data plane
- [Environments](/docs/docker-host/environments)  -  per-env bucket names
