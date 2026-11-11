---
slug: "api-reference/sandbox-env"
title: "沙盒环境 API"
description: "针对每个沙箱环境变量的 CRUD，静态加密。"
section: "API reference"
order: 12024
---

沙箱环境 API 管理附加到沙箱的环境变量的键值存储。变量在静态时进行加密，并在启动时注入沙箱进程环境中。 API 永远不会返回纯文本值 - 只返回键名称和元数据。

基本路径：`/api/v1/sandboxes/{id}/env`

> **注意**
>
> 变量名称必须匹配 `[A-Z][A-Z0-9_]*` - 大写字母、数字和下划线，以字母开头。这反映了 POSIX shell 环境变量约定。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/env` |列出所有环境变量（仅名称，无值）|
| `PUT` | `/api/v1/sandboxes/{id}/env` |批量设置或更新环境变量 |
| `DELETE` | `/api/v1/sandboxes/{id}/env/{key}` |删除单个环境变量 |

---

## 列出环境变量

**`GET /api/v1/sandboxes/{id}/env`**

返回所有变量名称。值永远不会返回。

### 授权

需要 `sandboxes:read` 范围。

```http
Authorization: Bearer msk_...
```

### 响应 - `200 OK`

```json
{
  "data": [
    { "key": "DATABASE_URL", "encrypted": true },
    { "key": "API_SECRET", "encrypted": true },
    { "key": "LOG_LEVEL", "encrypted": false }
  ]
}
```

---

## 设置环境变量

**`PUT /api/v1/sandboxes/{id}/env`**

批量更新插入。更新现有密钥；新密钥已创建。请求中不存在的密钥保持不变。

### 授权

需要 `sandboxes:write` 范围。

```http
Authorization: Bearer msk_...
```

### 请求正文

```json
{
  "vars": [
    { "key": "DATABASE_URL", "value": "postgres://...", "encrypted": true },
    { "key": "LOG_LEVEL", "value": "debug" }
  ]
}
```

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `vars[].key` |字符串|是的 |变量名称 (`[A-Z][A-Z0-9_]*`) |
| `vars[].value` |字符串|是的 |变量值 |
| `vars[].encrypted` |布尔 |没有 |静态存储加密（默认：`true`）|

### 响应 - `200 OK`

```json
{
  "data": [
    { "key": "DATABASE_URL", "encrypted": true },
    { "key": "LOG_LEVEL", "encrypted": false }
  ]
}
```

---

## 删除环境变量

**`DELETE /api/v1/sandboxes/{id}/env/{key}`**

删除单个变量。

### 响应 - `200 OK`

```json
{
  "deleted": true,
  "name": "LOG_LEVEL"
}
```

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `INVALID_ID` |沙箱 ID 不是有效的 UUID |
| 400 | `invalid vars` | `vars` 缺失或密钥名称验证失败 |
| 404 | `NOT_FOUND`|沙盒不存在或租户错误 |
| 404 | `NOT_FOUND`|键不存在（删除时）|

---

## 示例

**curl**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "vars": [
      {"key": "DATABASE_URL", "value": "postgres://user:pass@host/db"},
      {"key": "LOG_LEVEL", "value": "info"}
    ]
  }'

# List
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer msk_u_..."

# Delete
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env/LOG_LEVEL \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set
client.sandboxes.env.set("sbx_abc123", {
    "DATABASE_URL": "postgres://user:pass@host/db",
    "LOG_LEVEL": "info",
})

# List (names only)
keys = client.sandboxes.env.list("sbx_abc123")

# Delete one
client.sandboxes.env.delete("sbx_abc123", "LOG_LEVEL")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
await client.sandboxes.env.set('sbx_abc123', {
  DATABASE_URL: 'postgres://user:pass@host/db',
  LOG_LEVEL: 'info',
});

// List
const { data } = await client.sandboxes.env.list('sbx_abc123');

// Delete
await client.sandboxes.env.delete('sbx_abc123', 'LOG_LEVEL');
```
