---
slug: "api-reference/filesystem"
title: "文件系统API"
description: "MIOSA 计算机上文件和目录操作的完整 API 参考 - 上传、下载、stat、mkdir、重命名、复制、chmod 和 readdir。"
section: "API reference"
order: 12020
---

文件系统 API 使您能够以完全编程方式访问正在运行的计算机内的文件和目录。所有操作都通过计算机命令服务进行智能体，并限制在允许的路径上。

基本路径：`/api/v1/computers/{id}/files`

**允许的路径：** `/workspace`、`/home`、`/root`、`/tmp`、`/opt`、`/srv`

> **注意**
>
> 计算机必须处于 `"running"` 状态。允许前缀之外的路径返回 `403 FORBIDDEN_PATH`。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Write a file
await client.files.write(computerId, {
  path: '/workspace/hello.py',
  content: 'print("Hello from MIOSA")',
});

// List the directory
const { files } = await client.files.list(computerId, '/workspace');

// Download it back
const content = await client.files.download(computerId, '/workspace/hello.py');
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/files/upload` |上传文件（多部分）|
| `POST` | `/files/write` |直接写入文本/二进制内容 |
| `GET` | `/files` |列出目录 (`readdir`) |
| `GET` | `/files/download` |下载文件 |
| `POST` | `/files/export` |带元数据下载 (base64) |
| `DELETE` | `/files` |删除文件 |
| `GET` | `/files/stat` |统计文件或目录|
| `POST` | `/files/mkdir` |创建目录|
| `POST` | `/files/rename` |重命名或移动文件/目录 |
| `POST` | `/files/copy` |复制文件 |
| `POST` | `/files/chmod` |更改文件权限 |

---

## 上传文件

**`POST /api/v1/computers/{id}/files/upload`**

分段上传 - 将其用于二进制文件或大型有效负载。

### 请求 - `multipart/form-data`

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `file` |文件|是的 |要上传的文件 |
| `path` |字符串|没有 |远程路径。如果以 `/` 结尾，则附加文件名。默认值：`/workspace/` |

### 响应 - `201 Created`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/script.py",
    "filename": "script.py",
    "size_bytes": 1234
  }
}
```

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_FILE` |请求中没有文件字段 |
| 403 | `FORBIDDEN_PATH`|允许目录之外的路径 |
| 413 | `FILE_TOO_LARGE`|文件超过 10 MB 限制 |
| 502 | `AGENT_UNAVAILABLE`|虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/upload \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -F "file=@./script.py" \
  -F "path=/workspace/scripts/"
```

---

## 写入文件

**`POST /api/v1/computers/{id}/files/write`**

写入文本或 Base64 编码的二进制内容。对于编程使用来说，比 multipart 更快。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |完整目的地路径 |
| `content` |字符串|是的 |文件内容（`encoding="base64"` 时为 UTF-8 文本或 base64）|
| `encoding` |字符串|没有 | `"utf8"`（默认）或 `"base64"` |

### 响应 - `201 Created`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/config.json",
    "size_bytes": 256
  }
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/write \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/config.json", "content": "{\"key\": \"value\"}"}'
```

---

## 列出目录（readdir）

**`GET /api/v1/computers/{id}/files`**

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `path` |字符串|没有 |要列出的目录。默认值：`/workspace` |

### 响应 - `200 OK`
```json
{
  "files": [
    {
      "name": "Documents",
      "type": "directory",
      "size_bytes": 4096,
      "permissions": "drwxr-xr-x",
      "modified_at": "2026-04-11T10:30:00Z"
    },
    {
      "name": "script.py",
      "type": "file",
      "size_bytes": 1234,
      "permissions": "-rw-r--r--",
      "modified_at": "2026-04-11T10:25:00Z"
    }
  ],
  "path": "/workspace"
}
```
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 下载文件

**`GET /api/v1/computers/{id}/files/download`**

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `path` |字符串|是的 |文件的完整路径 |

### 响应 - `200 OK`

二进制文件内容。 `Content-Disposition` 设置为浏览器下载。

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_PATH` |无路径参数|
| 403 | `FORBIDDEN_PATH`|允许目录之外的路径 |
| 404 | `FILE_NOT_FOUND`|文件不存在 |
| 502 | `AGENT_UNAVAILABLE`|虚拟机内智能体无法访问 |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/download?path=/workspace/output.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output output.txt
```

---

## 导出文件

**`POST /api/v1/computers/{id}/files/export`**

在单个 JSON 响应中返回元数据和 Base64 编码的内容。当您需要将文件及其元数据放在一起而不处理二进制响应时非常有用。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |文件的完整路径 |

### 响应 - `200 OK`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/data.json",
    "filename": "data.json",
    "size_bytes": 5678,
    "content_type": "application/json",
    "modified_at": "2026-04-11T10:30:00Z"
  },
  "content_base64": "eyJrZXkiOiAidmFsdWUifQ=="
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/export \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/data.json"}'
```

---

## 删除文件

**`DELETE /api/v1/computers/{id}/files`**

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |要删除的完整路径 |

### 响应 - `200 OK`
```json
{ "success": true, "path": "/workspace/old-file.txt" }
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/old-file.txt"}'
```

---

## 统计文件或目录

**`GET /api/v1/computers/{id}/files/stat`**返回元数据而不下载文件内容。

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `path` |字符串|是的 |完整路径|

### 响应 - `200 OK`
```json
{
  "path": "/workspace/script.py",
  "type": "file",
  "size_bytes": 1234,
  "permissions": "-rw-r--r--",
  "owner": "user",
  "group": "user",
  "modified_at": "2026-04-11T10:25:00Z",
  "created_at": "2026-04-11T09:00:00Z"
}
```

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 404 | `FILE_NOT_FOUND`|路径不存在|
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/stat?path=/workspace/script.py" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 创建目录

**`POST /api/v1/computers/{id}/files/mkdir`**

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |要创建的目录路径 |
| `recursive` |布尔 |没有 |如果缺少则创建父目录（默认值：`false`）|

### 响应 - `201 Created`
```json
{ "success": true, "path": "/workspace/project/src" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/mkdir \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/project/src", "recursive": true}'
```

---

## 重命名或移动

**`POST /api/v1/computers/{id}/files/rename`**

重命名文件或目录，或将其移动到不同的路径。源和目标都必须在允许的路径内。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `source` |字符串|是的 |当前路径|
| `destination` |字符串|是的 |目标路径 |

### 响应 - `200 OK`
```json
{ "success": true, "source": "/workspace/old.txt", "destination": "/workspace/new.txt" }
```

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 404 | `FILE_NOT_FOUND`|源路径不存在 |
| 409 | `DESTINATION_EXISTS`|目的地已存在 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/rename \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "/workspace/app.py", "destination": "/workspace/app_v2.py"}'
```

---

## 复制文件

**`POST /api/v1/computers/{id}/files/copy`**

将文件复制到新路径。不支持目录复制 - 复制单个文件。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `source` |字符串|是的 |源文件路径 |
| `destination` |字符串|是的 |目标文件路径 |

### 响应 - `201 Created`
```json
{ "success": true, "source": "/workspace/template.py", "destination": "/workspace/project/main.py" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/copy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "/workspace/template.py", "destination": "/workspace/project/main.py"}'
```

---

## 更改权限

**`POST /api/v1/computers/{id}/files/chmod`**

设置 UNIX 文件权限。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |文件路径 |
| `mode` |字符串|是的 |八进制权限字符串，例如`"755"` 或 `"644"` |

### 响应 - `200 OK`
```json
{ "success": true, "path": "/workspace/script.sh", "mode": "755" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/chmod \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/script.sh", "mode": "755"}'
```

---

## 路径安全

所有路径在执行前都会经过验证：

1. **扩展** - `Path.expand` 解析 `..`、`~` 和符号链接
2. **前缀检查** - 必须以 `/workspace`、`/home`、`/root`、`/tmp`、`/opt` 或 `/srv` 开头
3. **拒绝** - 这些前缀之外的路径返回 `403 FORBIDDEN_PATH`

`/workspace/../../etc/passwd` 扩展为 `/etc/passwd` 并被拒绝。

---

## 常见食谱

### 搭建项目目录
```typescript
// Create structure
await client.files.mkdir(computerId, { path: '/workspace/app/src', recursive: true });
await client.files.mkdir(computerId, { path: '/workspace/app/tests', recursive: true });

// Write files
await client.files.write(computerId, { path: '/workspace/app/src/main.py', content: mainPy });
await client.files.write(computerId, { path: '/workspace/app/requirements.txt', content: reqs });

// Make entrypoint executable
await client.files.chmod(computerId, { path: '/workspace/app/src/main.py', mode: '755' });
```

### 检索构建工件
```python
# List the build output directory
files = client.files.list(computer_id, path="/workspace/app/dist")
for f in files["files"]:
    if f["type"] == "file":
        content = client.files.export(computer_id, path=f"/workspace/app/dist/{f['name']}")
        save_artifact(f["name"], content["content_base64"])
```

---

## 沙箱文件系统

沙箱原语在 `/api/v1/sandboxes/{id}` 下公开相同的列表和统计操作。

### 列出文件（沙箱）

**`GET /api/v1/sandboxes/{id}/files?path=/workspace`**

列出沙箱目录中的文件和目录。
```bash
curl "https://api.miosa.ai/api/v1/sandboxes/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

回应：
```json
{
  "data": {
    "path": "/workspace",
    "entries": [
      {"name": "index.html", "path": "/workspace/index.html", "is_dir": false, "size": 128}
    ]
  }
}
```

### 统计文件（沙箱）

**`POST /api/v1/sandboxes/{id}/files/stat`**

返回一个文件或目录的元数据。
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/files/stat \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path":"/workspace/index.html"}'
```
