---
slug: "develop/sandboxes/capabilities"
title: "能力"
description: "沙箱的用途、沙箱的具体用途以及核心执行、文件、预览、快照和生命周期功能。"
section: "Sandboxes"
order: 2001
---

此页面是[沙箱概述](/docs/develop/sandboxes)背后的参考：人们使用沙箱构建什么，沙箱实际上是什么，以及每个沙箱公开的功能。

## 沙箱的用途

- **智能体构建循环**：生成代码、编辑文件、安装依赖项、运行测试并将日志流回智能体。
- **预览服务器**：在沙箱中运行 Web 服务器并仅将所需端口公开为 HTTPS 预览 URL。
- **安全执行**：将不受信任的命令与后端隔离，同时将文件系统状态保持在沙箱范围内。
- **可重现状态**：快照工作状态，稍后恢复，或从已知点分叉以进行并行智能体工作。

## 具体来说，什么是沙盒

- 代码生成、构建、测试和开发服务器的独立计算。
- 命名、版本化的 CPU、内存和磁盘合约以及每个沙箱的超时和环境参数。
- `/workspace` 是可编辑的文件系统根目录。
- 从版本化模板启动，通常是`miosa-sandbox`。
- 默认情况下是持久的，因此 `timeout_sec` 停止计算并保留状态而不是删除工作区。
- 默认情况下网络隔离。
- 预览 URL 仅打开您公开的特定端口。
- 销毁是永久性的并删除已保存的状态。

## 核心能力

|能力|它给你什么 |
|---|---|
| `exec.run` |使用 stdout、stderr、退出代码和持续时间阻止命令执行 |
| `exec.stream` |用于长时间构建或安装的实时 stdout/stderr 事件 |
| `files.write/read/list/stat` |在 `/workspace` 和其他允许的路径下直接访问文件系统 |
| `previews.create` |特定沙箱端口上正在运行的进程的公共 HTTPS URL |
| `snapshot` |用于恢复、分叉或回滚的持久检查点 |
| `pause` / `resume` |停止计算而不丢弃保存的状态 |

## 看看它是否有效

**打字稿**

```typescript
const sandbox = await miosa.sandboxes.create({ templateId: "miosa-sandbox" })

await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)

const result = await sandbox.exec("python3 --version")
console.log(result.stdout)

const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
sandbox = miosa.sandboxes.create(template_id="miosa-sandbox")

sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)

result = sandbox.exec("python3 --version")
print(result.stdout)

preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```
