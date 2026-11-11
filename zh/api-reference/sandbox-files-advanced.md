---
slug: "api-reference/sandbox-files-advanced"
title: "沙盒文件（高级）API"
description: "上传文件、写入字节和文本、遍历文件树、批量写入以及监视沙箱中的文件更改事件。"
section: "API reference"
order: 12022
---

这些高级文件操作通过面向 IDE 的功能扩展了核心 [Files API](/docs/api-reference/files)：递归树列表、原子批量写入和实时文件更改事件流。

基本路径：`/api/v1/sandboxes/{id}/files`和`/api/v1/sandboxes/{id}/fs`为上传路由。

> **注意**
>
> 这些路由在内部使用 `:sandbox_id` 作为路径参数，但作为 `/sandboxes/{id}/files/...` 访问 - `{id}` 段是沙箱 UUID。

> **注意：关于 SOMA**
>
> 通过这些路径在 SOMA 沙箱上进行文件读取、写入、列出、mkdir 和删除工作。
> 他们不再回答`501 FILES_UNSUPPORTED_ON_RUNTIME`。
> SOMA 上的文件仅在沙箱运行时保留，因为 SOMA 尚未在停止时保持状态。
> 请参阅[选择运行时](/docs/soma/choosing-a-runtime)。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/files/tree` |递归文件树|
| `POST` | `/api/v1/sandboxes/{id}/files/write-many` |批量写入多个文件|
| `GET` | `/api/v1/sandboxes/{id}/files/watch` |文件更改事件的 SSE 流 |
| `POST` | `/api/v1/sandboxes/{id}/fs/upload` |上传文件（多部分）|

---

## 上传文件

**`POST /api/v1/sandboxes/{id}/fs/upload`**

将字节作为多部分表单数据发送。
这是 SDK 使用的路线，也是目前唯一的上传路线。

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `file` |文件|是的 |要写入的字节。缺少部分答案 `400 MISSING_FILE`。 |
| `path` |字符串|没有 |目的地。默认为 `/home/user/`。尾部斜杠附加上传的文件名；如果没有，`path` 就是确切的目的地。 |

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/fs/upload \
  -H "Authorization: Bearer msk_u_..." \
  -F "path=/workspace/assets/" \
  -F "file=@./logo.png"
```

### 错误

|状态 |代码|原因 |
|---|---|---|
| 400 | `MISSING_FILE` |没有 `file` 部分。 |
| 400 | `INVALID_PATH` | `path` 包含空字节。 |
| 413 | `FILE_TOO_LARGE` | 上传超出大小限制。 |

> **注意**
>
> `/api/v1/sandboxes/{id}/files/upload` 是该路由的别名。

---

## 写入字节，而不是文本

`/files/write` 逐字存储 `content`，因此它适用于文本。
对于任何二进制文件，请改为发送 `content_base64`； `/files/write-many` 按每个文件获取它。

```bash
# macOS uses `base64 -i`, GNU coreutils uses `base64 -w0`
B64=$(base64 -i ./logo.png | tr -d '\n')
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/files/write \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  --data "$(jq -n --arg p /workspace/logo.png --arg b "$B64" '{path: $p, content_base64: $b}')"
```

---

## 文件树

**`GET /api/v1/sandboxes/{id}/files/tree`**

返回递归目录列表，最多可配置深度。对于填充 IDE 文件资源管理器面板很有用。

### 授权

```http
Authorization: Bearer msk_...
```

### 查询参数

|参数|类型 |默认 |描述 |
|------------|------|---------|-------------|
| `path` |字符串| `/workspace` |要列出的根目录 |
| `depth` |整数 | `3` |最大遍历深度 (1-10) |
| `include_hidden` |布尔 | `false` |包括点文件和点目录 |

### 响应 - `200 OK`

```json
{
  "data": {
    "path": "/workspace",
    "type": "directory",
    "children": [
      {
        "name": "src",
        "path": "/workspace/src",
        "type": "directory",
        "children": [
          {
            "name": "index.ts",
            "path": "/workspace/src/index.ts",
            "type": "file",
            "size": 1842,
            "modified_at": "2026-05-26T09:12:01Z"
          }
        ]
      },
      {
        "name": "package.json",
        "path": "/workspace/package.json",
        "type": "file",
        "size": 512,
        "modified_at": "2026-05-25T18:00:00Z"
      }
    ]
  }
}
```

---

## 批量写入

**`POST /api/v1/sandboxes/{id}/files/write-many`**

在单个请求中写入多个文件。从来宾的角度来看，所有写入都以原子方式应用 - 要么全部成功，要么为失败的文件返回错误。

> **注意**
>
> 每个请求最多 50 个文件。大于 90 KB 的文件应通过 `POST /files/write` 单独上传。

### 请求正文

```json
{
  "files": [
    {
      "path": "/workspace/src/index.ts",
      "content": "export const hello = () => 'world';\n",
      "encoding": "utf8"
    },
    {
      "path": "/workspace/src/utils.ts",
      "content": "ZXhwb3J0IGZ1bmN0aW9uIG5vb3AoKSB7fQo=",
      "encoding": "base64"
    }
  ]
}
```

| 字段 | 类型 | 必填 | 描述 |
|--------|------|----------|-------------|
| `files[].path` |字符串|是的 |沙箱内的绝对路径 |
| `files[].content` |字符串|是的 |文件内容（UTF-8 文本或 base64 编码的二进制）|
| `files[].encoding` |字符串|没有 | `"utf8"`（默认）或 `"base64"` |

### 响应 - `200 OK`

```json
{
  "data": {
    "written": 2,
    "paths": [
      "/workspace/src/index.ts",
      "/workspace/src/utils.ts"
    ]
  }
}
```

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `INVALID_BODY` | `files` 缺失或不是数组 |
| 400 | `TOO_MANY_FILES` |一次请求超过 50 个文件 |
| 400 | `INVALID_PATH` |路径包含空字节或遍历沙箱外 |

---

## 文件监视（SSE）

**`GET /api/v1/sandboxes/{id}/files/watch`**

将文件系统更改事件作为 SSE 进行流式传输。对于实时同步编辑器状态很有用。

### 授权

```http
Authorization: Bearer msk_...
```

（浏览器 `EventSource` 无法设置此标头 - 使用服务器端客户端或作用域会话令牌。）

### 查询参数

|参数|类型 |默认 |描述 |
|------------|------|---------|-------------|
| `path` |字符串| `/workspace` |观看目录 |

### 响应 - `200 OK` (`text/event-stream`)

```text
id: wev_001
event: file.changed
data: {"path":"/workspace/src/index.ts","event":"modified","ts":"2026-05-26T10:14:23Z"}

id: wev_002
event: file.changed
data: {"path":"/workspace/src/new-file.ts","event":"created","ts":"2026-05-26T10:14:24Z"}

: heartbeat
```

**`data.event` 的事件值：**

| 值 | 触发条件 |
|-------|---------|
| `created` | 新的文件或目录 |
| `modified` | 文件内容发生变化 |
| `deleted` | 文件或目录被移除 |
| `renamed` | 文件被移动或重命名 |

---

## 示例

**curl**

```bash
# File tree (depth 2)
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/tree?depth=2" \
  -H "Authorization: Bearer msk_u_..."

# Batch write
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/write-many \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {"path": "/workspace/hello.ts", "content": "console.log(\"hello\");\n"}
    ]
  }'

# File watch (SSE)
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/watch?path=/workspace/src"
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Tree
tree = client.sandboxes.files.tree("sbx_abc123", path="/workspace", depth=3)

# Batch write
client.sandboxes.files.write_many("sbx_abc123", [
    {"path": "/workspace/index.ts", "content": "export {};"},
    {"path": "/workspace/utils.ts", "content": "export function noop() {}"},
])
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// File tree
const tree = await client.sandboxes.files.tree('sbx_abc123', {
  path: '/workspace',
  depth: 3,
  includeHidden: false,
});

// Batch write
await client.sandboxes.files.writeMany('sbx_abc123', [
  { path: '/workspace/index.ts', content: 'export {};' },
  { path: '/workspace/utils.ts', content: 'export function noop() {}' },
]);
```
