---
slug: "develop/files-and-exec"
title: "文件与执行"
description: "在沙箱内读取、写入、监视文件。运行命令、流式传输标准输出/标准错误、管理进程。"
section: "Sandboxes"
order: 2005
---

文件和执行 API 是您（或 AI 智能体）与沙箱交互的方式。它们共同构成了其他一切的基础：生成代码、运行构建、启动服务器。

## 文件

### 写

**打字稿**

```typescript
await sandbox.files.write("/workspace/src/index.ts", `
  console.log("Hello from MIOSA")
`)

// Binary
await sandbox.files.write("/workspace/logo.png", pngBuffer, {
  encoding: "binary",
})
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/files/write \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "path": "/workspace/src/index.ts",
    "content": "console.log(\"Hello from MIOSA\")"
  }'
```

父目录根据需要创建。现有文件将被覆盖。

### 阅读
```typescript
const content = await sandbox.files.read("/workspace/package.json")
const parsed = JSON.parse(content)
```
```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/files/read?path=/workspace/package.json \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### 导出生成的工件

当智能体创建用户应从您的网站下载的文件时，请使用导出
产品：HTML、PDF、CSV、ZIP、屏幕截图、报告或构建文件夹。

**打字稿**

```typescript
const exportInfo = await sandbox.createExport({
  paths: ["/workspace/dist/index.html", "/workspace/dist/assets/app.js"],
  label: "homepage build",
})

console.log(exportInfo.archiveDownloadUrl)

const archive = await sandbox.downloadExport([
  "/workspace/dist/index.html",
  "/workspace/dist/assets/app.js",
])
```

**Python**

```python
export_info = sandbox.create_export(
    ["/workspace/dist/index.html", "/workspace/dist/assets/app.js"],
    label="homepage build",
)

print(export_info["archive_download_url"])

archive = sandbox.download_export([
    "/workspace/dist/index.html",
    "/workspace/dist/assets/app.js",
])
```

**命令行界面**

```bash
miosa files cp sb_abc123:/workspace/dist/index.html ./index.html --json

miosa files cp -r sb_abc123:/workspace/dist ./homepage-build --json
```

对于一个文件，导出会返回该文件的直接下载 URL。对于多个
文件，导出返回存档下载 URL。这是相同的低级模型
复制或下载沙箱文件时：沙箱仍然有效
文件系统，并且您的产品决定哪些路径成为用户可见的工件。

将此模式用于工件产品：

```text
1. Agent writes files under /workspace/artifacts or /workspace/dist.
2. Your backend calls createExport with the exact paths users should receive.
3. Your UI stores the returned export descriptor on the task or run.
4. User clicks Download.
5. Your backend streams the direct file or tar archive from MIOSA.
```

示例：

|智能体输出|导出路径 |
|---|---|
|登陆页面 HTML | `/workspace/dist/index.html` |
|内置应用程序文件夹| `/workspace/dist` |
| PDF报告| `/workspace/artifacts/report.pdf` |
|销售 CSV | `/workspace/artifacts/leads.csv` |
|截图集| `/workspace/artifacts/screenshots` |
|生成的项目| `/workspace` 或过滤后的构建文件夹 |

不要让用户从终端输出中抓取文件。如果一次运行产生
可交付成果，将其转换为导出并将其附加到运行、任务或项目
记录在您的产品中。

对于智能体平台，将沙箱文件系统视为工作面并
导出为面向用户的合约。智能体可以创建许多中间体
`/workspace` 下的文件，但您的后端应仅提升批准的路径
进入命名的工件。

|产品图样 |工作路径 |出口产物|
|---|---|---|
| Lovable/Replit/Genspark 风格的应用程序构建器 | `/workspace`、`/workspace/dist` |源 ZIP、构建文件夹、预览、部署 URL |
| Polsia 式运算符 | `/workspace/research`, `/workspace/artifacts` |报告、CSV、营销活动副本、生成的应用程序包 |
| coFounder.co风格的公司操作系统| `/workspace/agents`、`/workspace/receipts`、`/workspace/artifacts` |里程碑文件、决策、状态报告 |
|Nebula式虚拟设备|分享`/workspace`加电脑截图|选定的文件、屏幕截图、日志、设备收据 |

存储导出 ID、标签、路径、大小、MIME 类型（如果可用）并下载
您自己的跑步记录上的 URL。这为用户提供了稳定的工件历史记录，即使
沙箱稍后会暂停、分叉或继续生成新文件。

### 列表
```typescript
const entries = await sandbox.files.list("/workspace")
for (const e of entries) {
  console.log(e.type, e.path, e.size)
}
```
```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/files/list?path=/workspace \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### 观看

流文件系统事件：
```typescript
for await (const event of sandbox.files.watch("/workspace")) {
  console.log(event.type, event.path)
  // event.type: "create" | "modify" | "delete"
}
```

监视被实现为服务器发送的事件。有关 SSE 事件形状，请参阅[事件](/docs/api-reference/events)。

### 文件系统布局

|路径|目的|
|---|---|
| `/workspace` |工作目录；这是智能体编写应用程序代码的地方 |
| `/home/sandbox` |用户首页 |
| `/tmp` |临时划痕（在沙箱销毁时清除）|
| `/opt/venv` |预装 Python virtualenv（`miosa-sandbox` 和 Python 模板）|
| `/usr/local/bin` |系统二进制文件|

一般来说，写在 `/workspace` 里面并让你的构建工具处理
node_modules、virtualenvs 等。生产发布冻结 `/workspace`
（不包括缓存目录）作为源快照。参见
[发布](/docs/docker-host/publishing)。

## 执行

在沙箱内运行命令。

### 一击

**打字稿**

```typescript
const result = await sandbox.exec("npm install", {
  cwd: "/workspace",
  env: { NODE_ENV: "development" },
  timeoutSec: 120,
})

console.log(result.exit_code, result.stdout, result.stderr)
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "cmd": "npm",
    "args": ["install"],
    "cwd": "/workspace",
    "timeout_ms": 120000
  }'
```

阻塞直到命令退出（或 `timeoutMs` 过去）。返回 `exit_code`、`stdout`、`stderr` 和 `duration_ms`。

### 长时间运行的服务器
```typescript
// Keep the process attached and consume its output as it runs.
const events = sandbox.exec.stream("node /workspace/server.js")
for await (const event of events) console.log(event)
```

使用 [Previews](/docs/develop/previews) 公开其端口。

### 流媒体

对于实时标准输出/标准错误（例如尾部智能体的编译输出）：
```typescript
for await (const chunk of sandbox.exec.stream("pnpm dev")) {
  if (chunk.type === "stdout") process.stdout.write(chunk.data)
  if (chunk.type === "stderr") process.stderr.write(chunk.data)
  if (chunk.type === "exit") console.log("exit:", chunk.exit_code)
}
```

作为SSE实施。请参阅[Streaming Exec](/docs/api-reference/streaming-exec)。

### 终端

需要交互式 PTY（用于远程开发或驱动 CLI 的 AI 智能体）？
```typescript
const session = await sandbox.terminal.create({})
// Attach over the session's WebSocket channel. See API Reference: Terminal for the protocol.
```

终端基于 WebSocket，具有 `xterm.js` 兼容的 ANSI 直通。有关协议详细信息，请参阅 [API 参考：终端 ](/docs/api-reference/streaming-exec)。

## 流程管理`exec` 阻塞，直到命令退出； `exec.stream` 在您消耗输出时保持其运行。要检查或停止进程，请在沙箱内运行 shell 命令：

```typescript
await sandbox.exec("ps aux")
await sandbox.exec("kill <pid>")
```

## 幂等性

Exec **不是**幂等的。运行 `npm install` 两次会运行两次。如果你
需要幂等重试，存储一个标记文件：
```typescript
const marker = "/workspace/.deps-installed"
const exists = await sandbox.files.exists(marker)
if (!exists) {
  await sandbox.exec("npm ci")
  await sandbox.files.write(marker, new Date().toISOString())
}
```

文件写入是幂等的：相同的路径、相同的内容、相同的结果。

## 限制

|限制|默认 |可调|
|---|---|---|
|每次写入的最大文件大小 | 500 MB |是（取决于计划）|
|每个沙箱的最大文件数 | 1,000,000 |是的 |
|最大执行标准输出缓冲区 | 10 MB |流模式绕过了这个 |
|最大执行持续时间 | `timeout_ms`参数，默认30秒；通过 `120000` 进行 npm/pip 安装 |是的;超然无极限|
|最大并发分离作业 | 100 | 是的|

> **注意：没有原始外壳逃避问题**
>
> `cmd` 和 `args` 作为元组直接传递给 `execve`，不带 shell
> 插值。您不需要引用参数或转义特殊字符。
> 如果您确实想要 shell，请设置 `cmd: "bash"` 并
> `args: ["-c", "&lt;your shell command&gt;"]`。

## 另请参阅

- [Sandboxes](/docs/develop/sandboxes)：您正在操作的虚拟机
- [预览](/docs/develop/previews)：公开通过 exec 启动的端口
- [API 参考：Streaming Exec](/docs/api-reference/streaming-exec)：SSE 有线格式
- [API 参考：Files](/docs/api-reference/files)：文件 API 传输格式
