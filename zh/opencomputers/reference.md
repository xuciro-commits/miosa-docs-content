---
slug: "opencomputers/reference"
title: "API、CLI 和 SDK 参考"
description: "每个 OpenComputers 端点组、调用它们的 CLI 命令系列以及包装它们的 SDK 命名空间。"
section: "Computers"
order: 3025
---

OpenComputers 的所有内容均可通过三种方式访问​​：REST API、`miosa` CLI 和 SDK。
此页是地图。主机、作业、文件、隧道、智能体和集群的字段级详细信息位于 [OpenComputers API 参考 ](/docs/api-reference/open-computers) 中。

## 休息 API

基本路径：`/api/v1/opencomputers`。所有调用都需要`Authorization: Bearer <platform-api-key>`。

|集团|端点 |
|---|---|
|主持人 | `GET /hosts`、`POST /hosts`、`GET /hosts/:id`、`DELETE /hosts/:id` |
|主机标签 | `PATCH /hosts/:id/tags`、`POST /hosts/:id/tags/:tag`、`DELETE /hosts/:id/tags/:tag`、`GET /tags` |
|主机运行时 | `GET /hosts/:id/runtime`、`POST /hosts/:id/runtime`、`POST /hosts/:id/runtime/certify`、`POST /hosts/:id/runtime/enrollment` |
|主机信息 | `GET /hosts/:id/metrics`、`GET /hosts/:id/services`、`GET /hosts/:id/jobs` |
|团体 | `GET /groups`、`POST /groups`、`GET /groups/:id`、`PATCH /groups/:id`、`DELETE /groups/:id`、`POST /groups/:id/members`、`POST /groups/:id/members/:host_id`、`DELETE /groups/:id/members/:host_id`、`POST /groups/:id/exec`、`POST /groups/:id/apps/:app_id/install` |
|工作机会 | `POST /hosts/:id/exec`、`GET /hosts/:id/exec`、`POST /hosts/:id/jobs`、`GET /hosts/:id/exec/:job_id`、`DELETE /hosts/:id/exec/:job_id` |
|文件| `GET /hosts/:id/fs`、`GET /hosts/:id/fs/stat`、`GET /hosts/:id/fs/download`、`POST /hosts/:id/fs/upload`、`DELETE /hosts/:id/fs`、`POST /hosts/:id/fs/mkdir` |
|终端和桌面 | `POST /hosts/:id/terminal/ticket`、`POST /hosts/:id/desktop/ticket` |
|集装箱 | `GET, POST /hosts/:host_id/containers`、`GET, PATCH, DELETE /hosts/:host_id/containers/:id`、`POST .../:id/start`、`POST .../:id/stop`、`POST .../:id/restart` |
|撰写| `GET, POST /hosts/:host_id/compose-projects`、`GET, DELETE /hosts/:host_id/compose-projects/:id`、`POST .../up`、`POST .../down` |
| IDE 会话 | `GET, POST /hosts/:host_id/ide-sessions`、`GET, DELETE /hosts/:host_id/ide-sessions/:id`、`POST .../reveal-password` |
| GitHub Actions 跑步者 | `GET, POST /hosts/:id/gha-runners`、`GET, DELETE /hosts/:id/gha-runners/:runner_id`、`POST .../refresh` |
|网格| `GET, POST /meshes`、`GET, DELETE /meshes/:id`、`POST /meshes/:id/hosts/:host_id`、`DELETE /meshes/:id/hosts/:host_id` |
|备份| `GET, POST /hosts/:host_id/backup-configs`、`PATCH, DELETE /hosts/:host_id/backup-configs/:config_id`、`POST .../pause`、`POST .../resume`、`POST .../trigger`、`GET .../snapshots` |
|快照| `GET /snapshots/:id`、`DELETE /snapshots/:id`、`POST /snapshots/:id/restore` |
|隧道 | `GET, POST /hosts/:host_id/tunnels`, `GET, PATCH, DELETE /hosts/:host_id/tunnels/:id` |
|隧道域 | `GET, POST /tunnels/:tunnel_id/domains`、`GET, DELETE /tunnels/:tunnel_id/domains/:id`、`POST .../verify` |
|秘密| `GET, POST /secrets`、`GET, POST /hosts/:id/secrets`、`PATCH, DELETE /hosts/:id/secrets/:secret_id`、`POST .../reveal` |
| SSH 密钥 | `GET, POST /hosts/:id/ssh-keys`、`DELETE /hosts/:id/ssh-keys/:key_id`、`POST /hosts/:id/ssh-keys/import/github` |
|剪贴板| `POST /hosts/:id/clipboard`、`GET /hosts/:id/clipboard` |
|应用程序 | `GET /apps`、`GET /hosts/:id/apps`、`POST /hosts/:id/apps/:app_id/install`、`GET /hosts/:id/apps/installs/:install_id`、`DELETE /hosts/:id/apps/:app_id`、`POST .../:app_id/start`、`POST .../:app_id/health` |
|工作空间 | `GET /workspaces`、`GET, POST /hosts/:host_id/workspaces`、`GET, PATCH, DELETE /hosts/:host_id/workspaces/:wid`、`POST .../pull`、`POST .../open-terminal`、`POST .../run`、`POST .../expose` |
|时间表| `GET, POST /hosts/:host_id/schedules`、`GET, PATCH, DELETE /hosts/:host_id/schedules/:id`、`POST .../pause`、`POST .../resume`、`POST .../trigger`、`GET .../runs` |
|股票 | `GET, POST /hosts/:id/shares`, `PATCH, DELETE /hosts/:id/shares/:share_id` |
|警报 | `GET, POST /alerts/rules`、`PATCH, DELETE /alerts/rules/:id`、`POST .../fire-test`、`GET, POST /alerts/channels`、`PATCH, DELETE /alerts/channels/:id`、`POST .../verify`、`GET /alerts/fires`、`POST /alerts/fires/:id/ack` |
|网络钩子 | `GET, POST /webhooks`、`PATCH, DELETE /webhooks/:id`、`POST .../test`、`GET .../deliveries` |
|审计| `GET /audit`、`GET /audit/export`、`GET /audit/verify`、`GET /audit/:id` |
|集群| `GET, POST /clusters`、`GET, DELETE /clusters/:id`、`POST /clusters/:id/start`、`POST /clusters/:id/stop` |

### 流端点

这些响应服务器发送的事件并保持连接打开：

|流 |端点|
|---|---|
|作业输出 | `GET /hosts/:id/exec/:job_id/stream` |
|主机生命周期 | `GET /hosts/events` |
|集装箱日志| `GET /hosts/:host_id/containers/:id/logs` |
|集装箱统计 | `GET /hosts/:host_id/containers/:id/stats` |
|撰写事件 | `GET /hosts/:host_id/compose-projects/:id/events` |
|网格事件| `GET /meshes/:id/events` |
|跑步活动 | `GET /hosts/:id/gha-runners/:runner_id/events` |
|智能体会话事件 | `GET /hosts/:id/agent/sessions/:session_id/events` |
|应用程序安装事件 | `GET /hosts/:id/apps/installs/:install_id/events` |
|集群事件 | `GET /clusters/:id/events` |
|工作空间活动 | `GET /hosts/:host_id/workspaces/:wid/events` |
|审计提要 | `GET /audit/live` |

`GET /hosts/events` 使用票据而不是不记名令牌：使用 `POST /api/v1/opencomputers/hosts/events/sse-ticket` 铸造一张票据，然后使用 `?ticket=` 连接。

### 推论

活动集群服务于 OpenAI 兼容端点：

|端点|目的|
|---|---|
| `POST /inference/:slug/v1/chat/completions` |聊天完成 |
| `POST /inference/:slug/v1/completions` |文本完成 |
| `GET /inference/:slug/v1/models` |型号列表 |

### 未经身份验证的端点

|端点|目的|
|---|---|
| `POST /opencomputers/hosts/registration-status` |检查主机密钥是否有效并已连接 |
| `POST /opencomputers/schedules/trigger/:token` |使用一次性令牌从 Webhook 触发计划 |
| `GET /opencomputers/osa/latest` |最新智能体发布清单 |
| `ANY /t/:slug/*path` |公共隧道智能体；隧道自身的接入方式决定谁能通过|

## 命令行界面

CLI 将 OpenComputers 公开为 `miosa host`，并使用 `miosa host` 作为别名。

**主人和工作**

```bash
miosa host register <name> --platform linux|macos|windows
miosa host list --tag <tag>
miosa host get <host>
miosa host metrics <host>
miosa host runtime <host>
miosa host services <host>
miosa host jobs <host>
miosa host job list|get|cancel <host> [job]
miosa host fs ls|stat|mkdir|rm <host> --path <path>
miosa host tag list|add|rm
miosa host group list|get|new|add|remove|exec|rm
miosa host secret list|set|rm
miosa host ssh-key list|add|import|rm
miosa host app catalog|list|install|start|health|uninstall
miosa host agent runs|run|cancel
miosa host rm <host> --yes
```

**容器和网络**

```bash
miosa host container list|get|run|start|stop|restart|rm
miosa host container logs|stats <host> <container>
miosa host compose list|up|down|rm
miosa host mesh list|new|get|add-host|remove-host|events|rm
miosa host tunnel list|get|new|rm
miosa host gha-runner list|new|get|events|refresh|rm
miosa host backup list|new|update|trigger|pause|resume|rm
miosa host backup snapshot list|get|fork|rm
miosa host cluster list|get|start|stop|rm
```

**任何运行时类型**

```bash
miosa device list
miosa device get|capabilities <device>
miosa device exec <device> -- <command>
miosa device files|read|write <device> --path <path>
miosa device expose <device> <port>
miosa device browser <device>
miosa device extend|pause|resume|stop|rm <device>
```

`miosa host` 和 `miosa device` 都接受全局标志（`--json`、`--org`、`--profile`、`--timeout` 等）。有关完整命令和标志列表，请参阅 [CLI 参考](/docs/cli/platform#your-own-hosts)。

## SDK

| SDK |命名空间|
|---|---|
| [TypeScript](/docs/sdks/typescript) | `client.openComputers` |
| [Python](/docs/sdks/python) | `client.open_computers` |
| [转到](/docs/sdks/go) | `client.OpenComputers` |
| [长生不老药](/docs/sdks/elixir) | `Miosa.OpenComputers.*` 模块 |

```typescript
import Miosa from '@miosa/sdk';

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

const host = await miosa.openComputers.hosts.create({ name: 'studio', platform: 'linux' });
console.log(host.host_key);                       // shown once

const job = await miosa.openComputers.jobs.start(host.id, { command: 'uname -a' });
console.log(job.links.stream);                   // stream this job
```

```python
import miosa

client = miosa.Miosa(api_key="msk_...")

for host in client.open_computers.hosts.list().data:
    print(host.id, host.status)
```

> **注意：SDK 不覆盖同一表面**
>
> 覆盖范围因语言而异。 Python 和 Elixir SDK 涵盖了最广泛的表面 - 容器、组合、网格、备份、运行器配置、计划、共享、警报、审计和 Webhooks。 TypeScript 和 Go SDK 涵盖主机、作业、文件、隧道、智能体、集群和机密。
> 当您的语言不包装资源时，请直接调用 REST 端点。

> **警告：某些方法是按设计提出的**
>
> 主机重命名在租户 API 上不可用，因此主机的 SDK `update` 方法会引发不受支持的错误，而不是发送请求。重命名只能在管理界面上进行。请改用标签对主机进行分组。

- [OpenComputers API 参考](/docs/api-reference/open-computers)：主机、作业、文件、隧道、智能体、集群和机密的请求字段和响应形状。

- [CLI 参考](/docs/cli/platform#your-own-hosts)：每个 miosa 主机和 miosa 设备命令。

- [Devices](/docs/opencomputers/devices): id 寻址的设备表面。
