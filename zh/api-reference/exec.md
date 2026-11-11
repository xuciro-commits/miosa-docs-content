---
slug: "api-reference/exec"
title: "执行API"
description: "用于在 MIOSA 计算机上执行 bash 命令和 Python 代码的 API 参考。"
section: "API reference"
order: 12018
---

直接在计算机上运行命令和脚本，无需桌面交互。两个端点都会阻塞，直到命令退出并返回组合的 stdout+stderr。

基本路径：`/api/v1/computers/{id}/exec`

> **注意**
>
> 计算机必须处于`"running"`状态。命令以虚拟机内的默认用户身份运行。 bash 执行超时上限为 86,400 秒； Python 执行超时上限为 300 秒。

> **注意**
>
> 速率限制因密钥、租户策略和端点系列而异。尊重 `X-RateLimit-*` 和 `Retry-After` 响应标头。对于长时间运行的输出，首选流式执行端点，它可以避免在整个持续时间内保持 HTTP 连接打开。

---

## 执行 Bash 命令

**`POST /api/v1/computers/{id}/exec`**

在计算机上运行 shell 命令。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `command` |字符串|是的 |执行的 Shell 命令 |
| `timeout` |整数 |没有 |超时以秒为单位（默认值：30，最大值：86400）|

### 响应  -  `200 OK`
```json
{
  "output": "total 24\ndrwxr-xr-x 6 user user 4096 Apr 11 10:00 .\n",
  "exit_code": 0
}
```

`output` 字段包含组合的 stdout 和 stderr。

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 409 | `COMPUTER_NOT_RUNNING`|计算机未运行 |
| 502 | -|虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "ls -la /workspace", "timeout": 30}'
```

### 示例
```bash
# Install a package
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "sudo apt-get install -y jq", "timeout": 120}'

# Check disk space
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "df -h"}'

# Multi-line script
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "cd /workspace && mkdir -p project && echo done"}'
```

---

## 执行Python代码

**`POST /api/v1/computers/{id}/exec/python`**

在计算机上运行 Python 代码。代码被写入临时文件并使用 `python3` 执行。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `code` |字符串|是的 | Python源代码|
| `timeout` |整数 |没有 |超时以秒为单位（默认值：30，最大值：300）|

### 响应  -  `200 OK`
```json
{
  "output": "42\n",
  "exit_code": 0
}
```

### 它是如何运作的

1. 代码写入`/tmp/miosa_exec_&lt;random&gt;.py`
2. 通过`timeout &lt;N&gt; python3 /tmp/miosa_exec_&lt;random&gt;.py 2&gt;&1`执行
3. 执行后删除临时文件
4. stdout 和 stderr 被捕获在 `output` 中

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 409 | `COMPUTER_NOT_RUNNING`|计算机未运行 |
| 502 | -|虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import sys\nprint(f\"Python {sys.version}\")\nprint(2 + 2)",
    "timeout": 30
  }'
```

### 示例
```bash
# JSON processing
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import json\ndata = {\"hello\": \"world\"}\nprint(json.dumps(data, indent=2))",
    "timeout": 10
  }'

# File processing
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import os\nfor f in os.listdir(\"/workspace\"):\n    print(f)",
    "timeout": 10
  }'

# Long-running computation
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "total = sum(range(10_000_000))\nprint(f\"Sum: {total}\")",
    "timeout": 60
  }'
```

---

## 终端

### 创建终端会话

**`POST /api/v1/computers/{id}/terminal`**

在计算机上创建基于 WebSocket 的终端会话。

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `cols` |整数 |没有 |终端列（默认：80）|
| `rows` |整数 |没有 |端子行（默认：24）|
| `shell` |字符串|没有 |要使用的 shell（默认值：`/bin/bash`）|

#### 响应  -  `201 Created`
```json
{
  "data": {
    "session_id": "default",
    "ws_url": "wss://my-computer.sandbox.miosa.ai/ws/terminal/550e8400-.../default?auth=stream_token",
    "stream_auth": "stream_token",
    "expires_at": 1712700060,
    "computer_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/terminal \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cols": 80, "rows": 24, "shell": "/bin/bash"}'
```

---

### 调整终端大小

**`POST /api/v1/computers/{id}/pty/{session_id}/resize`**

调整活动终端会话的大小。

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `cols` |整数 |是的 |新列数 |
| `rows` |整数 |是的 |新行计数 |

#### 响应  -  `200 OK`
```json
{
  "status": "ok"
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/pty/{session_id}/resize \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cols": 120, "rows": 40}'
```

---

## 超时行为

- 超过超时的命令将被 SIGTERM 终止
- bash 执行超时上限为 **86,400 秒**
- Python 执行超时被限制：`min(requested, 300)`
- 如果未指定超时，则默认为 **30 秒**

## 预装软件

默认模板包括：

- Python 3 与 pip
- Node.js（如果通过 selected_apps 安装）
- 常用CLI工具：curl、wget、git、vim、jq
- 系统实用程序：htop、tree、zip/unzip

通过 bash exec 端点安装附加包：
```bash
# Python packages
{"command": "pip install requests pandas numpy"}

# System packages
{"command": "sudo apt-get install -y postgresql-client"}
```

---

---

## 沙箱执行

`/api/v1/sandboxes/{id}` 下的沙箱存在相同的执行表面。

> **注意：SOMA 命令大小**
>
> 在 SOMA 沙箱上，可执行文件和每个参数都限制为 4096 字节，并且更长的 shell 命令会回答 `502 AGENT_UNAVAILABLE`，这看起来是可重试的，但实际上并非如此。
> 将命令作为文件发送，或者通过标准输入发送。
> 请参阅[选择运行时](/docs/soma/choosing-a-runtime)。

### 执行命令（沙箱）

**`POST /api/v1/sandboxes/{id}/exec`**

在沙箱内运行 shell 命令。阻塞直到进程退出。

#### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` |字符串|沙箱 ID。必须处于 `running` 状态。 |

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `command` |字符串|是的 |要执行的 shell 命令。 |
| `timeout` |整数 |没有 |每个命令的超时时间（以秒为单位）。默认 `3600`，最大 `86400`。 |
| `cwd` |字符串|没有 | VM 内的工作目录。 |
| `env` |对象|没有 |仅用于此调用的环境变量。 |

#### 响应  -  `200 OK`
```json
{
  "data": {
    "sandbox_id": "sbx_01j9xr2t4fk8me3n5q",
    "stdout": "2\n",
    "stderr": "",
    "exit_code": 0
  }
}
```

#### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 409 | `SANDBOX_NOT_RUNNING`|沙盒不处于 `running` 状态。 |
| 504 | -|命令超时。 |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "python3 -c \"print(1+1)\"", "timeout": 30}'
```

### 打开终端会话（沙箱）

**`POST /api/v1/sandboxes/{id}/terminal`**

在运行的沙箱内创建 PTY 会话并返回短暂的流
浏览器 WebSocket 客户端的令牌。

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `cols` |整数 |没有 |初始终端宽度。默认为 `80`。 |
| `rows` |整数 |没有 |初始终端高度。默认为 `24`。 |
| `shell` |字符串|没有 |外壳命令。可用时默认登录 bash。 |

#### 响应  -  `201 Created`
```json
{
  "session_id": "52d1df5a0a3a0018",
  "ws_url": "wss://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/terminal/stream",
  "stream_auth": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...",
  "stream_auth_expires_at": "2026-05-13T12:30:00Z"
}
```

使用以下命令打开 WebSocket：
```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/terminal/stream?session_id={session_id}&cols=120&rows=32&token={stream_auth}
```

客户端应以二进制帧的形式发送原始终端输入。发送调整大小事件
作为 JSON 文本框架：
```json
{"type":"resize","cols":120,"rows":32}
```

完成后删除会话：
```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/{id}/terminal/{session_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 另请参阅

- [Streaming Exec](/docs/api-reference/streaming-exec) - 用于长时间运行命令的 SSE 流
- [计算机 API](/docs/api-reference/computers) - 计算机 CRUD 和生命周期
- [Files API](/docs/api-reference/files) - 无需执行即可读写文件
- [错误代码](/docs/api-reference/errors) - `COMPUTER_NOT_RUNNING`、`AGENT_UNAVAILABLE`
