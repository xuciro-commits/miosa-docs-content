---
slug: "opencomputers/meshes"
title: "网格"
description: "在私有加密 WireGuard 网络上链接多台主机，以便它们无需通过公共互联网即可相互访问。"
section: "Computers"
order: 3017
---

网格是跨多个主机的命名专用网络。
MIOSA 在网格的 `/24` 中为每个成员分配一个地址并分发 WireGuard 密钥，因此成员可以直接相互联系，而不会在公共互联网上暴露任何内容。

当一台主机上的服务需要与另一台主机上的服务通信时，请使用网格：一个主机上的数据库，第二个主机上的应用程序，第三个主机上的 GPU 工作线程。

## 创建网格

**MIOSA CLI**

```bash
miosa host mesh list
miosa host mesh new lab --host <host> --host <host>
miosa host mesh get lab
```

别名：`miosa host meshes`。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/meshes" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"lab","host_ids":["<host-1>","<host-2>"]}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `name` |字符串|是的 | 1 到 80 个小写字母数字或连字符 |
| `host_ids` |字符串[] |没有 |创建时添加的主机；默认 `[]` |

网格带有 `10.x.x.0/24` 形式的 `cidr`、`state` 和默认为 `51820` 的 `listen_port_base`。
网格状态为 `provisioning`、`active`、`failed` 和 `torn_down`。

## 管理会员资格

```bash
miosa host mesh add-host lab <host>
miosa host mesh remove-host lab <host> --yes
miosa host mesh rm lab --yes
```

|行动|休息 |
|---|---|
|添加主机 | `POST /opencomputers/meshes/{id}/hosts/{host_id}` |
|删除主机 | `DELETE /opencomputers/meshes/{id}/hosts/{host_id}` |
|删除网格 | `DELETE /opencomputers/meshes/{id}` |

添加主机会返回新的对等点。
删除主机后，主机本身仍保留在原位；删除网格会在删除记录之前破坏 WireGuard 配置。

> **注：253 个主机的网状上限**
>
> 网格的 `/24` 地址空间允许 253 个成员。添加一个过去，返回 `422` 和 `mesh /24 is full (max 253 hosts)`。
> 如果您无法满足单个网格的需求，请联系支持人员，而不是自行嵌套网格。

## 检查同行

`miosa host mesh get <mesh>` 和 `GET /opencomputers/meshes/{id}` 返回网格及其当前对等列表。

|同行领域|意义|
|---|---|
| `id` |对等记录 ID |
| `host_id` |该对等点代表的主机 |
| `assigned_ip` |网格的 `10.x.x.x` 范围内的地址 |
| `pubkey` |对等方的 WireGuard 公钥 |
| `endpoint` |最后看到的端点 |
| `latest_handshake` | 隧道最后一次完成握手是什么时候 |
| `rx_bytes`、`tx_bytes` |客流量计数器|
| `state` | `pending_key`、`configured`、`active` 或 `failed` |

对等状态是回答“该主机实际上是否可以通过网格到达”的最快方法：最近有一个 `latest_handshake` 的 `active` 对等点是，并且 `pending_key` 对等点从未加入。

## 活动

**MIOSA CLI**

```bash
miosa host mesh events lab
```

流式传输主机加入、密钥轮换和状态更改，直到您按 Ctrl-C。

**休息API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/meshes/$MESH_ID/events" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

命名的 SSE 事件：`mesh_peer_active`、`mesh_peer_down` 和 `mesh_status`。

## 安全说明

- 网状网络流量经过加密并保留在网状网络内。由于主机是成员，因此无法从 Internet 访问它。
- MIOSA 分发 WireGuard 密钥。具有控制平面访问权限的操作员可以将主机添加到网格中，因此将网格成员资格视为特权更改并在[审核日志](/docs/opencomputers/security#audit-log)中查看它。
- 网格不会授予主机访问另一台主机上超出主机防火墙和服务允许的端口的任何其他内容的权限。
