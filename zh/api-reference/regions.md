---
slug: "api-reference/regions"
title: "区域 API"
description: "列出可用区域并将计算机和沙箱固定到特定区域。"
section: "API reference"
order: 12013
---

MIOSA 计算机和沙箱在地理区域中运行。每台计算机（以及每台沙箱，因为沙箱是一种轻量级计算机风格）都是在一个区域中创建的，并在其生命周期内保留在那里。

基本路径：`/api/v1/regions`

---

## 概述

- **今天有 3 个区域上线。** 所有区域都具有平等的地位 - 没有一个是预览版，没有一个受到限制。
- **默认区域**（当您未指定时）：`us-west`。
- **所有虚拟机尺寸**（XS、S、M、L、XL）在每个区域均适用于计算机和沙箱。
- 对于美国用户来说，**延迟**大致是等距的 - 根据数据驻留或邻近偏好而不是性能进行选择。
- **没有多区域资源。** 一台计算机仅位于一个区域。要在多个区域运行工作负载，请创建多台计算机。

---

## 可用区域

|蛞蝓 |地点 |城市 |状态 |
|------------------|--------------------|------------------|--------|
| `us-west` |美国西部|加利福尼亚州洛杉矶 |直播 |
| `us-east` |美国东部 |纽约州纽约 |直播 |
| `us-mia` |美国东部 2 |迈阿密 |直播 |

区域 slug 是您在创建时传递的内容以及在读取时返回的内容 - 每个计算机和沙箱响应都包含一个 `region` 字段，例如`"region": "us-west"`。

---

## 创建时指定区域

创建计算机或沙箱时，在请求正文中传递 `region`。如果省略，则使用默认值 (`us-west`)。

**Python**

```python
  computer = client.computers.create(
      name="ny-build-box",
      size="M",
      region="us-east",
  )
  print(computer.region)  # "us-east"
```

**打字稿**

```typescript
  const computer = await client.computers.create({
    name: 'ny-build-box',
    size: 'M',
    region: 'us-east',
  });
  console.log(computer.region); // "us-east"
```

**curl**

```bash
  curl -X POST https://api.miosa.ai/api/v1/computers \
    -H "Authorization: Bearer $MIOSA_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "name": "ny-build-box",
      "size": "M",
      "region": "us-east"
    }'
```

相同的 `region` 字段适用于 `POST /api/v1/sandboxes`。

### 错误

|状态 |错误 |原因 |
|--------------------|----------------------------------|------------------------------------------------|
| 400 | `invalid_region` |蛞蝓不在上面的列表中 |
| 409 | `region_unavailable`|暂时容量问题 - 重试或选择另一个 |

---

## 默认区域行为

如果创建请求中省略 `region`：

- 资源解析为租户的默认区域（默认情况下为 `us-west`；在仪表板设置中更改它 - 该覆盖适用于省略 `region` 的 API 调用）。
- 响应仍然包含已解析的 `region`，因此您始终知道资源所在的位置。

---

## 列出区域

**`GET /api/v1/regions`**

返回租户可用的区域。对于 UI 中的区域选择器很有用。

### 响应 - `200 OK`
```json
{
  "regions": [
    {
      "id": "us-west",
      "label": "US-West (Los Angeles)",
      "active_nodes": 2,
      "available": true,
      "small_slots_free": 70,
      "headroom": {
        "xs_slots_free": 250,
        "small_slots_free": 70,
        "medium_slots_free": 14,
        "large_slots_free": 7,
        "total_memory_mb": 524288,
        "used_memory_mb": 12288
      }
    },
    {
      "id": "us-east",
      "label": "US-East (New York)",
      "active_nodes": 1,
      "available": true,
      "small_slots_free": 40,
      "headroom": { "xs_slots_free": 120, "small_slots_free": 40, "medium_slots_free": 8, "large_slots_free": 4, "total_memory_mb": 262144, "used_memory_mb": 6144 }
    },
    {
      "id": "us-mia",
      "label": "US-East (Miami)",
      "active_nodes": 1,
      "available": true,
      "small_slots_free": 30,
      "headroom": { "xs_slots_free": 90, "small_slots_free": 30, "medium_slots_free": 6, "large_slots_free": 3, "total_memory_mb": 262144, "used_memory_mb": 4096 }
    }
  ]
}
```

### 响应字段

|领域|类型 |描述 |
|--------------------------------|---------|--------------------------------------------------------------------------------|
| `id` |字符串|稳定区域 slug - 在创建时将其作为 `region` 传递 |
| `label` |字符串|人类可读的标签（可以安全地在 UI 中呈现）|
| `active_nodes` |整数 |现已在该地区注册的主机|
| `available` |布尔 | `true` 当该区域具有至少一个 `small` 沙箱的容量时 |
| `small_slots_free` |整数 | `small` 形状上的空闲槽 |
| `headroom` |对象|每个大小的可用插槽，加上该区域的总内存和已用内存 |
```bash
curl https://api.miosa.ai/api/v1/regions \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

> **提示**
>
> 列表端点无需经过身份验证 - 您可以使用短期票证或公共令牌来填充区域选择器，而无需暴露您的 `msk_*` 密钥。

---

## 常见问题解答

### 为什么是这 3 个区域？

我们选择了洛杉矶、纽约和迈阿密来覆盖美国西海岸、东海岸和中部走廊。对于美国普通用户来说，这三个站点的距离大致相等，并且每个站点都拥有独立的网络和电力足迹，因此其中一个站点的区域性中断不会影响其他站点。

### 如何选择区域？

首先按**数据驻留**进行选择。如果您有将数据固定到海岸的客户或合规性要求，那么这由您决定。如果您没有居住要求，请根据 **与您的用户或 CI 运行者的距离**进行选择 - 较短的物理距离意味着 VNC 流和 PTY 往返的 RTT 较低。不同区域的计算性能本身是相同的。

### 一台计算机可以跨多个区域吗？

不会。一台计算机（或沙盒）在其生命周期中只生活在一个区域。要在多个区域运行工作负载：

- 每个区域创建一台计算机。
- 如果您想要跨它们的单个逻辑工作区，请使用 OpenComputers 联合（请参阅 [OpenComputers](/docs/api-reference/open-computers)）。

今天，地区之间没有实时迁移。

### 会有更多地区发货吗？

是的 - 欧盟和亚太地区已在路线图上。当它们发货时，它们将与当前三个一起出现在 `GET /api/v1/regions` 中。观看[变更日志](/docs/changelog)以获取公告。

---

## 另请参阅

- [计算机 API](/docs/api-reference/computers) - 创建时传递 `region`
- [沙箱 API](/docs/api-reference/sandboxes) - 沙箱区域支持
- [OpenComputers](/docs/api-reference/open-computers) - 跨区域联合 BYOC 主机
