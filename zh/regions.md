---
slug: "regions"
title: "区域"
description: "MIOSA 在哪些区域运行、哪些资源属于区域本地、如何将沙箱或计算机固定到某个区域，以及跨区域的情况。"
section: "API reference"
order: 12008
---

每台计算机和每个沙箱都只在一个区域中创建，并在其整个生命周期内留在该区域。资源不会跨区域，目前也不支持区域之间的实时迁移。

## 可用区域

| Slug | 位置 | 城市 | 状态 |
|---|---|---|---|
| `us-west` | 美国西部 | 加利福尼亚州洛杉矶 | 可用 |
| `us-east` | 美国东部 | 纽约州纽约市 | 可用 |
| `us-mia` | 美国东部 2 | 迈阿密 | 可用 |

三个区域均已全面开放，且均无访问限制。计算机默认使用工作区的默认区域；如果未设置，则使用 `us-mia`。沙箱没有默认区域，会放置在容量允许的位置。

## 将资源固定到区域

创建时传入 `region`；该字段对计算机和沙箱均适用。

**Python**

```python
computer = client.computers.create(name="ny-build-box", size="M", region="us-east")
print(computer.region)   # "us-east"
```

**TypeScript**

```typescript
const computer = await client.computers.create({ name: 'ny-build-box', size: 'M', region: 'us-east' });
```

如果省略 `region`，计算机会使用工作区的默认区域（回退到 `us-mia`），而沙箱会放置在容量允许的位置；设置区域后，响应中会包含解析后的区域。你可以在控制台设置中更改工作区的默认区域；该覆盖值会应用于创建时省略该字段的计算机。

| 状态 | 错误 | 原因 |
|---|---|---|
| 400 | `invalid_region` | Slug 不在上述列表中 |
| 503 | `REGION_UNAVAILABLE` | 临时容量问题；请重试或选择其他区域 |

## 列出区域

**`GET /api/v1/regions`** 返回你的租户可用的区域，因此 UI 中的区域选择器无需硬编码列表。

```bash
curl https://api.miosa.ai/api/v1/regions \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

每个条目包含 `id`（区域 slug）、`label`、`active_nodes`、`available`、`small_slots_free`，以及一个 `headroom` 对象，其中包含各规格的可用槽位和区域的总内存与已用内存。完整响应结构请参阅[区域 API](/docs/api-reference/regions)。

## 哪些属于区域本地

- **计算。** VM 在一个区域中运行。所有 VM 规格在每个区域都可用于计算机和沙箱。
- **数据。** 数据服务和卷位于使用它们的资源所在区域。
- **延迟。** 对美国用户的中位数而言，三个美国站点大致等距，因此计算性能实际上相同。优先按数据驻留要求选择；否则按与用户或 CI runner 的邻近程度选择，这可以降低桌面串流和终端往返的 RTT。

## 跨区域

区域之间不支持实时迁移。要在多个区域运行工作负载，请为每个区域创建一个资源；如果希望在这些资源之上形成一个逻辑工作区，可使用 [OpenComputers](/docs/api-reference/open-computers) 联邦自带主机。

欧盟和亚太区域已在路线图中，届时会与当前三个区域一起出现在 `GET /api/v1/regions` 中。

> **注意：区域不是 runner 端点**
>
> 资源的区域（`us-west`、`us-east`、`us-mia`）与提供快速创建和 exec 调用的选择启用 runner 端点是两回事。区域决定 VM 所在位置；runner 端点决定请求如何到达 MIOSA。两者的区别请参阅[区域与 runner](/docs/develop/sandboxes/sizing)。

- [区域 API](/docs/api-reference/regions)：区域列表和错误的请求与响应结构。
- [规格与限制](/docs/develop/sandboxes/sizing)：命名的资源合同和选择启用的 runner 快速路径。
