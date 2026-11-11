---
slug: "opencomputers/troubleshooting"
title: "OpenComputers 故障排除"
description: "诊断离线主机、拒绝的凭据、失败的命令、容器、网格、运行器、备份和隧道问题。"
section: "Computers"
order: 3024
---

从失败的确切主机 ID、组织和操作开始。
检查控制平面记录和机器本身。
有效的登录、健康的 API 端点或保存的主机密钥并不能证明主机可以执行您的请求。

最快的第一次检查总是相同的：

```bash
miosa host get <host>
miosa host jobs <host>
```

`state` 和 `connected` 现在描述；工作清单描述了实际发生的事情。

## 主机离线

1. 确认机器已唤醒且网络连接正常。
2、检查主机上的agent进程；对于 [连接您的第一个主机](/docs/opencomputers/connect) 中的 systemd 设置，请使用以下命令。
3. 检查端口 443 上对 `api.miosa.ai` 的出站 HTTPS 和 WebSocket 访问；仅 HTTP 运行状况检查并不能测试 WebSocket 智能体。
4. 确认服务以 `~/.osa/open_computers.toml` 包含预期主机配置的用户身份运行。
5. 检查主机是否未被撤销，然后在 MIOSA 中确认新的心跳。

```bash
systemctl status osa-opencomputers.service --no-pager
journalctl -u osa-opencomputers.service -n 50 --no-pager
```

如果您的帐户无法读取系统日志，请使用 `sudo` 作为日志。
`osa opencomputers status` 可以帮助检查配置，但其会话输出描述了该命令查询的运行时。
在得出单独的后台进程离线的结论之前，请先确认实际的服务和控制平面心跳。

## 密钥已保存但未连接

`osa opencomputers login` 保存主机密钥，`connect` 是该命令的别名。
智能体仍然必须以主机模式启动；仅保存凭据不会启动会话。
按照[启动步骤](/docs/opencomputers/connect#4-start-the-host-session)进行操作。
请勿用平台 API 密钥替换主机密钥。

## 重新安装后身份被拒绝

计算机的身份文件必须与主机首次连接时固定的身份匹配。
在升级期间保留 `~/.osa/open_computers.ed25519` 以及主机配置。
如果身份丢失，请撤销主机并注册新主机，而不是复制另一台机器的身份或禁用验证。

## 注册失败

|错误 |原因 |
|---|---|
| `402 plan_limit_reached` |已达到您计划的主机限制；响应携带 `limit` 和 `current` |
| `422 name is required` | `name` 缺失、为空或超过 64 个字符 |
| `501 open_computers_disabled` |此环境未启用 OpenComputers |

在发现错误之前，请在[计费和限制](/docs/opencomputers/billing)中检查计划的主机限额。

## 命令失败或卡住

|症状|检查什么 |
|---|---|
| `cmd is required` |将 `cmd` 字段发送到 exec 端点；`command` 是较新的 `POST .../jobs` 路由上的字段名，不是 `/exec` 上的字段名 |
| `409 host_not_connected` |主机离线。发货前先调出智能体 |
| `422 invalid_command` |命令为空或超过 8192 个字符 |
| `422 invalid_args` |超过 256 个参数或非字符串参数 |
| `422 invalid_timeout` |超时时间超出 1 到 3600 秒 |
| `502 dispatch_failed` |调度时会话中断；检查智能体，然后重试一次 |

- **权限被拒绝或命令不可用：** 作为智能体用户在本地运行相同的命令并检查其权限、工作目录、PATH 和支持的操作。
- **HTTP 请求保持打开状态：** 默认情况下执行流；检查 SSE 事件直到 `exec_result` 或 `exec_error`，或使用 `stream: false` 作为简短命令。
- **超时或连接丢失：** 保存返回的作业 ID，检查作业和本地进程，并在重试之前确定结果。

```bash
curl --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/exec/$JOB_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

超时或客户端断开连接并不能证明进程已停止。
避免仅仅因为客户端断开连接而重试部署、支付、文件突变或其他具有副作用的操作。

## 容器

|症状 |检查什么 |
|---|---|
| `503 host_not_connected` |容器运行时由主机上的智能体管理；离线主机无法创建或启动容器 |
| `402 plan_limit_reached` |容器和组合槽预算已满；请参阅[计费和限制](/docs/opencomputers/billing) |
|卡在`provisioning` |主机正在拉取映像，或者该主机的运行时无法拉取映像引用 |
| `409` 开始 |容器仅从 `stopped` 或 `failed` 开始；首先检查 `state` |
| `last_error` 设置 |运行时拒绝了容器；重试之前阅读 `last_error` |

通过慢速链接拉取大图像可能需要几分钟的时间。使用 `miosa host metrics <host>` 监视主机的磁盘和网络，而不是重新创建容器。

## 网格

|症状|检查什么 |
|---|---|
|对等方陷入 `pending_key` |主机尚未完成密钥交换；确认主机在线并且智能体是最新的 |
|对等 `active` 但流量失败 |另一台主机上的服务可能没有侦听您拨打的地址，或者主机防火墙阻止了网状接口 |
| `422 mesh /24 is full` |该网成员已达253人 |
| `422 one or more hosts not found` |主机 ID 属于另一个组织或不存在 |

使用 `miosa host mesh get <mesh>` 并读取 `latest_handshake`、`rx_bytes` 和 `tx_bytes` 来区分“从未连接”和“已连接但无流量”。

## GitHub Actions 跑步者

|症状|检查什么 |
|---|---|
| `422 GitHub API rejected the PAT` |令牌缺少存储库或组织运行者权限，或已过期 |
| `422 GitHub repo/org not found` | `repo_url` 错误，或者令牌看不到它。不支持 GitHub Enterprise Server URL |
| `429 GitHub API rate limit exceeded` | GitHub 限制了注册调用；等待并重试 |
|跑步者 `offline` |主机离线，或者运行程序进程死亡。使用 `miosa host gha-runner refresh <host> <runner>` |
|跑步者 `failed` |阅读跑步者赛事；安装失败通常是主机缺少依赖项 |

从不调度的工作流程通常是标签不匹配：检查工作流程的 `runs-on` 标签是否与运行程序的 `labels` 匹配。

## 备份

|症状|检查什么 |
|---|---|
| `503` 触发 |必须连接源主机才能拍​​摄快照 |
|快照 `failed` |读取快照上的`error`；路径消失或权限更改是常见原因 |
| `422 snapshot has no chunks` |快照未完成；仅`complete`快照恢复|
| `503 target host not connected` |恢复由目标主机的智能体推送，因此目标必须在线 |
| `403 cross-organization restore not allowed` |目标主机与快照不在同一组织中 |

## 隧道未到达应用程序

1. 在主机上，确认应用程序正在运行并在预期的本地端口上做出响应。
2. 检查隧道的配置`target_port`和主机的连接。
3、打开返回的`public_url`；不要自己构造主机名。
4. 确认查看者符合隧道的访问策略。

|智能体错误 |意义|
|---|---|
| `connection_refused` |主机上的 `target_port` 上没有任何内容正在侦听 |
| `target_port_not_allowed` |端口超出主机智能体允许的范围 |
| `tunnels_disabled_on_host` |该主机上的隧道已禁用 |
| `too_many_concurrent_tunnels` |主机已达到并发隧道限制 |
| `502 host_offline` |主机在请求进行时掉线 |
| `503 tunnel paused` |隧道处于状态 `paused` |
| `401` 与 `WWW-Authenticate: Basic` |隧道处于 `password` 模式并且请求未携带凭据 |

公共 URL 无法使不可用的本地服务保持健康。
不要仅仅为了解决身份验证问题而将私有隧道更改为公共隧道。

## 命令可以工作，但桌面不行

无头主机可以在不提供桌面的情况下执行命令。
验证所选主机宣传桌面支持并具有所需的会话和流后端。
检查智能体当前平台支持情况；注册 macOS 或 Windows 并不能保证桌面流正常工作。
对于 MIOSA 管理的计算机，请遵循 [桌面准备情况](/docs/computers/fast-start)，而不是在任意主机上安装桌面。

## 客户云资源存在，但放置不可用现有的云基础设施只是入门步骤之一。
在预期安置之前，请验证提供商的接受度、成功的实时金丝雀以及主机的准备情况。
请参阅 [BYOC 概述 ](/docs/infrastructure/byoc/overview) 了解适用的登机口。

## 寻求帮助时应包含的信息

包括主机 ID、操作或作业 ID、带时区的时间戳、智能体版本、操作系统和体系结构以及相关错误文本。
说明该命令是否以智能体用户身份在本地运行以及心跳是否新鲜。
从共享日志中删除 API 密钥、主机密钥、身份文件、包含令牌的私有 URL 以及敏感命令输出。

- [连接您的第一个主机](/docs/opencomputers/connect)：重复连接和完成的命令检查。

- [计费和限制](/docs/opencomputers/billing)：在将 402 视为故障之前检查计划限制。
