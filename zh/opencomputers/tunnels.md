---
slug: "opencomputers/tunnels"
title: "隧道和暴露服务"
description: "公开位于 MIOSA 管理地址的主机上的端口，选择谁可以访问该端口，并了解隧道可以做什么和不能做什么。"
section: "Computers"
order: 3020
---

隧道在 MIOSA 管理的地址的一台主机上发布一个端口。
主机不断拨出MIOSA； MIOSA 将入站请求路由回该会话到您的服务。

出站管理会话和可公开访问的应用程序隧道是不同的东西。
连接主机不会暴露任何内容。隧道是一个明确的、独立的暴露，具有自己的访问策略。

## 创建隧道

```bash
miosa host tunnel new <host> --port 3000 --slug my-app-dev --access organization
miosa host tunnel list <host>
miosa host tunnel get <host> <tunnel>
miosa host tunnel rm <host> <tunnel> --yes
```

|旗帜|意义|
|---|---|
| `--port int` |主机上要公开的端口 |
| `--slug string` |地址名称（默认：生成）|
| `--access string` |谁可以打开它：`organization`（默认）或`public` |

通过 REST 进行相同的调用：

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/tunnels" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"target_port":3000,"auth_mode":"tenant_only","slug":"my-app-dev"}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `target_port` |整数 |是的 |要公开的本地端口，`1` 到 `65535` |
| `auth_mode` |字符串|没有 | `public` \| `tenant_only` \| `password`（默认：`tenant_only`）|
| `slug` |字符串|没有 |地址名称，4 到 63 个小写字母数字或连字符 |
| `target_host` |字符串|没有 |主机从座席拨号（默认：`127.0.0.1`）|
| `password` |字符串|没有 |实际中当 `auth_mode` 为 `password` 时需要；散列，从未返回 |

```json
{
  "id": "tun_abc",
  "host_id": "host_abc123",
  "slug": "my-app-dev",
  "target_port": 3000,
  "auth_mode": "tenant_only",
  "public_url": "https://<public-host>/t/my-app-dev",
  "enabled": true,
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

使用返回的 `public_url`。不要自己构建主机名：路由域是根据环境设置的，因此从响应中读取它而不是硬编码。

## 更改或禁用隧道

```bash
miosa host tunnel new <host> --port 8080 --slug my-app-dev   # re-create with a new port or slug
```

通过 REST，`PATCH /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}` 就地更改隧道：

|领域|类型 |描述 |
|---|---|---|
| `state` |字符串| `active`、`paused` 或 `revoked` |
| `auth_mode` |字符串|更改访问模式 |
| `target_host` |字符串|更改智能体拨打的主机 |
| `slug` |字符串|更改地址名称 |
| `rotate_slug` |布尔 |生成一个新的 slug |
| `password` |字符串|设置新密码（重新哈希）|

`DELETE /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}` 返回 `204 No Content` 并撤销隧道。

该响应添加了可用于检查流量是否流动的实时计数器：`state`、`custom_domain`、`request_count`、`lifetime_bytes_ingress` 和 `lifetime_bytes_egress`。

## 访问模式

|模式|谁能打开|
|---|---|
| `organization` |仅限您的 MIOSA 组织的成员（CLI 默认） |
| `tenant_only` |拥有主机的租户 |
| `public` |知道 URL 的任何人（API 默认）|

> **注意：请勿开启私服进行故障排除**
>
> 公共 URL 无法使不可用的本地服务正常运行，也无法解决身份验证问题。
> 在更改隧道的访问模式之前，请确认主机自身端口上的服务应答。

## 暴露设备端口

`miosa device expose` 为任何设备类型（包括主机）发布端口：

```bash
miosa device expose <device> 3000
```

当您想要更完整的表面 - 列表、获取、访问模式和 slug 控制时，请使用键入的 `miosa host tunnel` 命令。

## 隧道不能做什么

- 它不会改变应用程序的所有者。您的服务仍然在您的机器上运行，并在您的流程监督下。
- 它不会消除服务监听的需要。隧道到达港口；它不会启动任何东西。
- 它不会使超出传输 MIOSA 终止的传输中的流量变为私有。设置访问模式和应用程序自身的身份验证以匹配您提供的数据。

- [API 参考中的隧道](/docs/api-reference/open-computers#tunnels)：请求字段、响应形状和访问配置。

- [安全模型](/docs/opencomputers/security)：出站会话授予什么以及不授予什么。

- [OpenComputers 故障排除](/docs/opencomputers/troubleshooting#tunnel-does-not-reach-the-application)：当隧道未到达应用程序时应检查哪些内容。
