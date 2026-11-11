---
slug: "troubleshooting"
title: "故障排查"
description: "处理沙箱、exec、预览、机密、网络和部署故障的入口，以及每类问题的排查位置。"
section: "Help"
order: 16011
---

先确认准确的资源 ID 和失败的操作。大多数请求还会返回 `x-request-id` 响应头——提交支持请求时请附上它。

```bash
miosa doctor   # checks CLI version, config, endpoint, key, and clock skew
```

时钟偏差会破坏签名 Webhook 和过期令牌，因此 `miosa doctor` 是很快的第一步检查。

## 沙箱无法启动

- **状态仍为 `provisioning`。** 运行时尚未就绪。轮询 `GET /api/v1/sandboxes/{id}`，等待 `state` 变为 `running`、`ready` 变为 `true`。
- **状态为 `error`。** 启动或运行时失败。检查 `sbx.data["metadata"]` 中的 `last_error`，然后销毁并新建一个沙箱。
- **`exec` 被拒绝。** 仍在启动的沙箱会拒绝命令。CLI 的 `exec` 会以退避方式重试，最长 90 秒；SDK 调用方也应这样处理，或等待 `ready`。

## 命令失败或挂起

- **命令返回非零状态。** 沙箱会以远程命令的状态码退出。请读取 `stderr`；不会有任何输出被静默吞掉。
- **`exec` 超时。** 沙箱 `exec` 端点接受 1 到 86,400 秒的 `timeout`（默认 3,600 秒），不受沙箱自身超时限制。长时间任务请启动[后台进程](/docs/develop/sandboxes/long-running-tasks)并轮询其日志。
- **文件上传失败。** 单次上传的文件上限为 500 MB。更大的产物请拆分，或从沙箱内部写入。
- **HTTP `429`。** 触发了速率限制。请按 `Retry-After` 响应头中的秒数等待。参见[速率限制](/docs/rate-limits)。

## 预览无法加载

- **`404` 或连接被拒绝。** 所暴露端口上没有进程在监听。确认服务器绑定到 `0.0.0.0`，而不只是 `127.0.0.1`，并确认端口号一致。
- **私有预览需要令牌。** 私有预览需要有效令牌。可见性和分享方式参见[预览](/docs/develop/previews)。

## 机密不生效

机密相关深入内容见[安全故障排查](/docs/security/troubleshooting)。该页面汇总了最常见的问题——占位符未被替换、范围未绑定或不匹配、轮换似乎没有生效——并给出根因和修复方法，同时涵盖 Connect Accounts、网络允许列表、审计日志和白标凭据。

## 网络被阻止

沙箱启动时处于仅审计模式，因此在执行允许列表之前不会阻止任何请求。如果调用被阻止，请检查待处理队列（`sandbox.network.pending()`），或按 `action=reject` 过滤审计日志；每一行都会在 `rejected_by` 中注明触发的规则。常见遗漏（`pip`、`npm`、CDN）列在[互联网访问和出口](/docs/develop/sandboxes/internet-access)中。

## 部署失败

- **构建失败。** 读取该部署版本的构建日志；容器构建路径参见 [Docker Host 部署指南](/docs/docker-host/docker-deploy)。
- **自定义域名无法解析。** 根域记录需要 DNS 提供商支持 ALIAS / ANAME。
- **需要恢复上一版本。** 回滚到先前的发布版本；参见[发布](/docs/docker-host/releases)和[回滚](/docs/docker-host/rollback)。

主机、隧道和自带云问题请参见 [OpenComputers 故障排查](/docs/opencomputers/troubleshooting)。

## SSH 和连接

没有公开的 SSH 端点。`miosa ssh`、SDK SSH 隧道和预览 URL 是仅有的接入方式。参见[连接到正在运行的沙箱](/docs/develop/sandboxes/connect)。

## 仍然无法解决

请从 MIOSA 控制台创建支持工单，并附上资源 ID、操作、带时区的时间戳和 `x-request-id`。粘贴内容前请移除 API 密钥、令牌和私有 URL。其他支持渠道参见[支持](/docs/support)。

- [安全故障排查](/docs/security/troubleshooting)：机密、OAuth、网络允许列表、审计日志和白标问题。
- [OpenComputers 故障排查](/docs/opencomputers/troubleshooting)：OpenComputer 连接、命令失败、桌面和隧道。
- [已知限制](/docs/known-limitations)：目前尚不支持的功能，以及有意不纳入范围的内容。
