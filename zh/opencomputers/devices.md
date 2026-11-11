---
slug: "opencomputers/devices"
title: "设备"
description: "设备模型 - 一个 ID 寻址的表面，用于运行命令、移动文件、公开端口以及跨运行时类型驱动生命周期。"
section: "Computers"
order: 3021
---

设备是可以在其上运行命令的任何运行时 MIOSA。
`<device>` 是一个 id，而不是名称，设备表面不关心哪种运行时答案。

当脚本必须跨运行时类型工作时，或者当您有一个 id 但不知道它是什么时，请使用它。

> **注意：设备和主机是不同的表面**
>
> `/api/v1/devices` 端点解析沙箱工作人员和托管计算机 - MIOSA 为您提供的运行时。
> 使用 OpenComputers 连接的计算机是 **主机**，并通过 [OpenComputers 端点 ](/docs/api-reference/open-computers) 通过其主机 ID 进行寻址。
> [`miosa device`](/docs/cli/devices)命令是CLI的统一视图；对于主机，请使用键入的 [`miosa host`](/docs/cli/platform#your-own-hosts) 系列。

## 列出并检查

**MIOSA CLI**

```bash
miosa device list
miosa device list --status running --name web --limit 20
miosa device get <device>
miosa device capabilities <device>
```

别名：`miosa devices`、`show` 代表 `get`，`caps` 代表 `capabilities`。

**休息API**

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices?status=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/capabilities" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

`GET /api/v1/devices` 接受 `status`、`name` 和 `limit`（默认 50）。

设备记录包含 `id`、`name`、`slug`、`template`、`kind`、`state`、`ip_address`、`node_id`、`running`、`terminal` 及其`capabilities`。

## 行动前检查能力

功能是在任何地方都能运行的脚本与在一半设备上失败的脚本之间的区别。

|能力|沙箱工作者|电脑|
|---|---|---|
| `exec` |是的 |是的 |
| `files` |是的 |是的 |
| `browser` |没有|是的 |
| `artifacts` |是的 |没有|
| `live_server` |是的 |是的 |
| `connectors` |是的 |是的 |
| `pause`、`resume` |是的 |是的 |
| `snapshots` |是的 |没有|

读取功能而不是从设备名称中假设它。

## 运行命令

**MIOSA CLI**

```bash
miosa device exec <device> -- uname -a
miosa device exec <device> -c "npm test" --timeout-ms 120000
```

`miosa device exec` 退出并显示远程命令的状态，如 `miosa exec`。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/exec" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command":"uname -a","timeout_ms":30000}'
```

正文需要 `command`（必需）和 `timeout_ms` 或 `timeout`，以秒为单位（默认 30）。

设备上的 Exec 解析 **正在运行** 目标。
未运行的设备会返回 `DEVICE_NOT_RUNNING` 以及当前的 `state`，而不是为您启动它。

## 文件

**MIOSA CLI**

```bash
miosa device files <device> --path /workspace
miosa device read <device> --path /workspace/out.json
miosa device write <device> --path /workspace/app.py --content "print(1)"
```

别名：`ls` 代表 `files`。

**休息API**

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files/read?path=/workspace/out.json" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/files/write" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path":"/workspace/app.py","content":"print(1)"}'
```

`files` 列出一个目录，默认为 `/workspace`。 `files/read` 接受必需的 `path` 并返回使用 `encoding: "base64"` 进行 base64 编码的内容。 `files/write` 采用 `path` 加上 `content_base64` 或 `content`。

`miosa device read` 返回 base64 编码的文件内容，`miosa device write` 从 `--content` 或带有 `--file` 的 JSON 正文获取内容。

## 暴露一个端口

```bash
miosa device expose <device> 3000

curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/devices/$DEVICE_ID/expose" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port":3000}'
```

在主机上，键入的 [隧道命令](/docs/opencomputers/tunnels) 为您提供更全面的信息：访问模式、slug 控制以及列表和删除。

## 浏览器网址

```bash
miosa device browser <device>
```

`GET /api/v1/devices/{id}/browser` 返回计算机的流式桌面，其中包含 `desktop_url`、`ws_url`、`token`、`expires_at`、`computer_id` 和 `slug` 以及一小时令牌 TTL。
沙箱返回 `BROWSER_UNSUPPORTED_FOR_SANDBOX`；当您需要桌面时，请使用[计算机](/docs/computer-use)。

## 生命周期

```bash
miosa device extend <device> --timeout 2h
miosa device pause <device>
miosa device resume <device>
miosa device stop <device>
miosa device rm <device> --yes
```

|行动|休息 |笔记|
|---|---|---|
|延伸| `POST /devices/{id}/extend` 与 `timeout_sec` |取代生存时间|
|暂停 | `POST /devices/{id}/pause` |保留磁盘|
|简历 | `POST /devices/{id}/resume` | |
|停止| `POST /devices/{id}/stop` |保留磁盘|
|摧毁| `DELETE /devices/{id}` |永恒的;返回状态 `destroyed` |

每个生命周期调用都会返回设备的新 `state`。

## 错误代码

|代码|状态 |意义|
|---|---|---|
| `MISSING_PARAM` | 400 |缺少必填字段；响应将其命名为 |
| `INVALID_PARAM` | 400 |字段存在但不可用 |
| `INVALID_BASE64` | 400 |文件内容的 base64 无效 |
| `NOT_FOUND` | 404 | 该设备不存在，或者不是您的|
| `DEVICE_NOT_RUNNING` | 409 | 设备未运行；响应携带其 `state`|
| `UNSUPPORTED_FOR_COMPUTER` | 422 | 该操作不适用于电脑|
| `BROWSER_UNSUPPORTED_FOR_SANDBOX` | 422 | 浏览需要电脑|
| `DEVICE_OPERATION_FAILED` | 422 | 运行时拒绝了操作|

- [计算机](/docs/computers/overview)：计算机设备背后的托管桌面运行时。
