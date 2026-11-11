---
slug: "services/replicas"
title: "服务副本与扩缩容"
description: "设置服务运行的副本数量、区域和规格，并了解扩缩容变更对运行中发布的影响。"
section: "Services & projects"
order: 1014
---

副本是服务的一个运行实例。
一个服务可运行 1 到 50 个副本。

## 更改副本数、区域和规格

```bash
miosa service update shop API --replicas 4
miosa service scale  shop API --replicas 4
```

`update` 更改设置。
`scale` 更改副本数、区域或限制，并会排队生成一个应用这些更改的发布。
在新发布变为活动状态之前，当前运行的发布不会受到影响，因此扩缩容变更绝不会中途中断流量。

| 设置 | 含义 |
|---|---|
| `replicas` | 1 到 50。存在可写卷时会强制为 1。 |
| `region` | `us-west`、`us-east` 或 `us-mia`。 |
| `limits` | `{vcpu, memory_mb}`。超过 4 vCPU 或 8192 MB 时，发布失败。 |
| `restart_policy` | `always`、`on_failure` 或 `never`。 |
| `healthcheck_path` | 新发布变为活动状态前，Web 服务必须响应的路径。 |

## 发布对副本的影响

新发布会先启动其副本。
副本通过健康检查后，新发布变为 `active`，较早的发布会逐步排空流量并停止。
`overlap_s` 和 `terminate_grace_s` 会增加较早发布停止前的等待时间。

## 卷

卷只能挂载到服务上，从不会单独添加。
可写卷只允许一个副本。
如需更多副本，请以只读方式挂载。
一个服务最多可挂载四个卷，路径和卷都必须唯一。

## 检查设置的生效情况

```bash
miosa service settings shop
```

每项设置的状态为 `enforced`、`stored` 或 `unsupported`。
`stored` 状态的设置会被保留，但尚未应用，请不要依赖它。

## 下一步

- [运行模式与类型](/docs/services/services)：所有设置及其适用的服务类型。
- [发布与回滚](/docs/services/releases)：扩缩容变更如何转化为发布。
- [日志](/docs/services/logs)：每个副本输出的内容。
