---
slug: "services/dedicated-vcpu"
title: "专用 vCPU"
description: "在每颗 vCPU 均为独立硬件线程且从不共享的配置上运行服务：规格、价格以及如何选择。"
section: "Services & projects"
order: 1015
---

专用规格让服务拥有完全独占的 vCPU。
每颗专用 vCPU 都是宿主机的一个硬件线程，固定分配给您的服务，绝不与其他工作负载共享。
当您需要稳定、不受其他工作负载争抢的 CPU 时，请使用它。

专用规格在 Business 及以上计划中提供。

## 规格与价格

内存为每颗 vCPU 4 GiB。

| 规格 | vCPU | 内存 | 每月价格 |
|---|---|---|---|
| `dedicated-2` | 2 | 8 GiB | $153.37 |
| `dedicated-4` | 4 | 16 GiB | $306.74 |
| `dedicated-8` | 8 | 32 GiB | $613.48 |
| `dedicated-16` | 16 | 64 GiB | $1,226.95 |

- 一颗专用 vCPU 每月 $32.8725（每小时 $0.04503），内存按常规的每 GiB-月 $10.953 计费。
- 无论 CPU 忙碌还是空闲，只要副本在运行，专用服务就按其完整规格逐秒计费。
- 已停止或休眠的服务不计计算费用。
- 同样数量的 vCPU 和内存使用共享规格时便宜约 17%，但没有相应保障。

## 选择规格

在服务设置中设置 `shape`。

```bash
curl -X PATCH https://api.miosa.ai/api/v1/projects/{id}/services/{sid} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"settings": {"shape": "dedicated-4"}}'
```

- 该规格应用于下一次部署。
- `replicas` 会成倍增加：三个 `dedicated-4` 副本使用 12 颗专用 vCPU。
- 如果您同时设置 `limits`，它们必须符合该规格。
- 查看 `GET /api/v1/projects/{id}/services/setting-capabilities`：当专用规格可供调度时，`shape` 条目显示为 `enforced`。

## 容量

每台提供专用 vCPU 的宿主机都有一个固定的 16 线程池，专用 vCPU 从中分配。
部署只会调度到这类宿主机上；当该宿主机的池中空闲线程不足时，部署会以 `no_capacity` 被拒绝。
16 颗 vCPU 的部署会占用整整一个宿主机池。
副本停止后，空闲线程会归还到池中。
