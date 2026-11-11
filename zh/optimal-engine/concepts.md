---
slug: "optimal-engine/concepts"
title: "Optimal Engine概念"
description: "Optimal Engine的对象是由租户、工作空间、节点、信号、源包、声明、事实、记忆、上下文包和投影构建的，以及源如何成为审查记忆。"
section: "Optimal Engine"
order: 8001
---

Optimal Engine将三件事分开：您提供的证据、从该证据中接受的真相以及它为读者构建的观点。
下面的每个对象都属于这三个对象之一。

## 拓扑结构

```text
Tenant / Organization
  -> Workspaces
      -> Nodes
          -> sources, signals, memories, wiki pages
```

|对象|它是什么 |
|---|---|
|租户/组织|顶级范围。它拥有工作区并保存每个工作区读取的组织策略。 |
|工作空间 |一个有围墙的部分，有自己的节点、来源、记忆、循环和节奏。一个密钥可以限制在某些工作区。 |
|节点|工作区中的打字内容。 |
|节点关系 |两个节点之间的类型化链接。 |

工作区 ID 为 `<tenant>:<slug>`，例如 `default:sales` 表示 `default` 租户中的工作区 `sales`。
工作区 `default` 始终存在。

节点种类，因为 `POST /api/nodes` 接受它们： `unit`、`team`、`project`、`entity`、`domain`、`department`、`operation`、`learning`、`person`、 `product`、`partnership`、`context`。

每个新工作区还获取设置和拓扑工具使用的标准节点类型：`entity`、`division`、`department`、`team`、`role`、`project`、`process`、`task`、`operation`、 `learning`、`person`、`customer`、`product`、`partnership`、`context`。

关系类型：`parent_child`、`depends_on`、`blocks`、`references`、`collaborates_with`、`sequence_before`、`instance_of`、`owns`、`participates_in`、`supports`。

项目只是一种节点。
将正在进行的工作（例如招聘或支持）建模为 `operation` 节点。

## 从来源到记忆

```text
source -> signal -> claim -> fact -> context package
```

|对象|它是什么 |
|---|---|
|源码包|原始输入与其元数据、信任标签和保留状态保持一致。它永远不会被重写。 |
|信号|引擎工作的机密单位。 |
|背景 |索引单元搜索返回：信号、资源、内存或技能文件。 |
|索赔 |引擎在源中找到的断言，以及返回它的链接。索赔正在等待审核，直至得到审核。 |
|事实|一项已被接受的主张，具有证据、可信度、准确性及其有效时间。 |
|内存对象|围绕事实、来源和时间的含义，例如聊天会话中的片段。 |
|派生账本|每个主张、事实和记忆来自哪个来源的记录。 |

### 信号

每个输入都分为五个不同的维度：

```text
Signal = Mode + Genre + Type + Format + Structure
```

将它们分开可以让引擎以不同于规范或代码文件的方式路由、解析和检索会议记录。
分类是基于规则的，无需语言模型即可运行。

### 主张和事实

引擎永远不会将提取的句子本身视为真实的。
主张通过以下两种方式之一成为事实：

- 拥有`claims:review`资助的人或关键人物可以推广它。
- 工作区升级规则自动升级。

规则用`OPTIMAL_CLAIM_PROMOTION_MODE`设置：

|模式|行为 |
|---|---|
| `auto_threshold`（默认）|提升分数达到工作区阈值（默认为 0.85）的声明 |
| `manual` |每项索赔都等待审核 |
| `agent_review` |目前的行为类似于 `manual` |

升级的声明将被保留并标记为 `promoted`，因此证据追踪保持完整。
一个新的事实可以取代一个旧的事实，而旧的事实仍然可以作为历史来阅读。

### 版本化记忆

除了声明生命周期之外，引擎还保留了一个更简单的内存存储，您可以使用 `POST /api/memory` 直接写入。
每个内存都有版本，并且内存通过 `updates`、`extends`、`derives`、`contradicts` 或 `cites` 相互链接。
忘记记忆只是隐藏它，而不是删除它；删除是一个单独的、永久的调用。

`POST /api/memory/remember` 是门控方式：它对新记忆进行新颖性和显着性评分，然后添加它、更新类似记忆、跳过它或将其作为主张提出。

## 读取内存

| 对象 | 它是什么 |
|---|---|
| 上下文包 | 一个读者在一次任务中可能看到的内容：所使用的事实和记忆、它们的来源、置信度和有效期。它是一种视图，而不是真相。 |
| 活动内存池 | 一次任务的工作记忆，以及在执行期间做出的观察。 |
| 检索策略 | `tiered`、`reconstructive` 或 `hybrid`。参见[内存与检索](/docs/optimal-engine/memory-and-retrieval)。 |

## 预测

引擎的数据库是事实的来源。
其他一切都是它的投影：

- **工作区文件**：每个工作区都有一个包含 `nodes/`、`.wiki/`、`assets/` 和 `architectures/` 的文件夹。在那里编辑文件会作为新的来源返回，而不是对事实的直接更改。
- **Wiki 页面**：带有引文的精选页面，保存在 `.wiki/` 中。
- **工作空间捆绑**：将整个工作空间导出为一个文件并导入到另一个引擎中。
- **API、CLI 和 MCP 服务器**：相同的状态，由人员和智能体读取。

## 治理

|对象|它是什么 |
|---|---|
| API 密钥 |识别呼叫者。它包含范围和它可能使用的工作空间列表。 |
|权威|允许、询问或拒绝读取、搜索、写入、导出、擦除和同步的规则。否认胜利。它以 `off`、`observe` 或 `enforce` 模式运行。 |
|组织政策|为组织设置一次的规则，每个工作区都会读取并且无法更改。 |
|安全标签和分区|行上的标签，例如 `confidential`，以及键可以读取的分区。 |

> **注意：API 中的单词**
>
> API 使用与此页面相同的名称：`workspace`、`node`、`claim`、`memory`、`context_package`。
> 结果中的 URI 使用 `optimal://` 方案。
