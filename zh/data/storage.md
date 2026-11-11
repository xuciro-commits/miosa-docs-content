---
slug: "data/storage"
title: "存储/桶"
description: "每个项目的对象存储，带有签名 URL 上传和下载。"
section: "Databases & storage"
order: 4006
---

> **提示：可用**
>
> 对象存储已发货。存储桶、对象、预签名 URL（上传和下载）、配额实施和每租户隔离都是实时的。大型直接上传（>5 MB）应使用预先指定的路径；请参阅下面的限制表。

## 你得到什么

每个项目：一个与 S3 兼容的存储桶，通过 MIOSA 端点访问。作为环境变量注入的凭据：
```bash

STORAGE_BUCKET=miosa-proj_xxx-uploads
STORAGE_ENDPOINT=https://storage.miosa.app
STORAGE_KEY_ID=...
STORAGE_KEY_SECRET          # 由 MIOSA 注入，请勿填写
```

您的代码使用标准 S3 客户端库：
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

## 按照您的方式使用：API、CLI 或 SDK

无需 SDK。同一个桶，三种方式。

> 存储调用需要具有 `storage:read` / `storage:write` 范围的 API 密钥。使用 `POST /api/v1/api-keys {"scopes": ["storage:read","storage:write"]}` 创建一个。普通密钥会收到 403 错误消息，告诉您缺少哪个作用域。

**原始API（curl）：**
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

**命令行界面：**
```bash
miosa storage create my-bucket
miosa storage list
miosa storage objects <bucket-id>
```

上传、下载、预签名没有专门的子命令；用 `miosa api` 驱动它们：

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

## 用于浏览器上传的签名 URL

不要通过运行时智能体文件上传。在服务器端创建一个签名 URL；浏览器直接上传到存储：
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

下载模式相同（`operation: "get"`）。签名的 URL 是有范围的（单密钥、单操作、短期到期）——没有将存储桶范围内的凭据泄露给浏览器的风险。

## 公共与私人

每个对象可以是私有的（默认；只能通过签名的 URL 访问）或公共的（从 `*.storage.miosa.app` URL 提供）。公共对象还可以通过项目的 [Domain](/docs/docker-host/domains) 设置以自定义域为前端。

## 带宽和存储成本

两个计费维度：

- **存储** - GB 月存储字节数。
- **出口** - GB 从 MIOSA 到公共互联网。 MIOSA 中的存储到运行时是免费的。

每个事件都会带有[归属](/docs/platform/attribution)以进行退款。

## 限制

|限制|默认 |可调|
|---|---|---|
|每个项目的存储| 1 GB |是（取决于计划）|
|直接API上传大小 | 5 MB 已验证 |较大的上传应使用预签名/直接存储 |
|最大物体尺寸 | 5 GB 目标 |多部分目标高达 100 GB |
|列出每次调用的键 | 1000 | 标准 S3 分页|

> **注意**
>
> 2026年6月9日最新实况存储烟雾已验证创建、上传、
> 通过 `10 / 10` 顺序 `5 MB` 下载、SHA-256 验证和清理
> 对象。 `8 MB` 和 `10 MB` 直接原始上传目前在存储之前失败，因此
> 大文件应使用预签名 URL 或 S3 兼容的直接上传流，直到
> 提升流API路径。请参阅[基准](/docs/benchmarks)了解
> 实测表和ComputeSDK存储比较。

## 沙箱也可以使用存储桶

相同的环境变量被注入沙箱中以进行开发/测试。从沙箱内上传：
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

## 带上自己的水桶吗？

是的。将环境变量设置为指向外部 S3 兼容存储桶（真正的 AWS、Backblaze B2、R2 等）。 MIOSA 会注入您设置的任何内容。

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

## 另请参阅

- [概述](/docs/data/overview) - 更广泛的数据平面
- [环境](/docs/docker-host/environments) - 每个环境的存储桶名称
