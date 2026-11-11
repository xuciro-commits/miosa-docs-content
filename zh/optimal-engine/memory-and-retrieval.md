---
slug: "optimal-engine/memory-and-retrieval"
title: "记忆与检索"
description: "Optimal Engine如何搜索（全文、矢量、混合）、询问和组合返回内容、如何将声明审查为事实、版本化内存如何工作以及基准数字的含义。"
section: "Optimal Engine"
order: 8006
---

Optimal Engine有四种读取内存的方法，从普通搜索到智能体的受控上下文包。

|致电 |返回|用它来 |
|---|---|---|
| `GET /api/search` |排名结果及其来源|找东西|
| `GET /api/grep` |使用意图、规模和模态过滤器匹配块 |查找准确的段落|
| `POST /api/rag` (`optimal ask`) |适合读者大小的上下文信封，包含来源|为模型提供回答一个问题所需的信息 |
| `POST /api/assemble` (`optimal assemble`) |包含经过审查的事实和记忆的分层上下文包 |在智能体采取行动之前向其通报情况 |

他们都没有用语言模型写出答案。
它们返回上下文及其来源；你的模型会写出答案。

## 搜索

```bash
curl -s "http://127.0.0.1:4200/api/search?q=renewal&workspace=default:notes&limit=10&mode=auto" \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

|参数|默认 |意义|
|---|---|---|
| `q` |必填|查询 |
| `workspace` | `default` |工作区 ID |
| `limit`、`offset` | `10`、`0` |寻呼 |
| `mode` | `auto` | `auto`、`lexical` 或 `hybrid` |
| `type` |全部 | `memory` 只搜索持久记忆 |
| `as_of` |现在 | ISO 8601 即时衡量新近度 |

每个结果都有一个 `id`、`title`、`uri`、`content`、一个短 `l0_abstract`、一个 `score` 以及它来自的 `workspace_id`。
`inherited: true` 标记来自组织策略而不是工作区本身的结果。

### 排名如何运作

- **全文**：SQLite FTS5 BM25，乘以新近度衰减（取决于源类型）和信号质量得分。
- **矢量**：当设置嵌入模型时，嵌入的余弦相似度。
- **混合**：两者的加权混合，然后增强图表中连接的结果。

| `mode` |行为 |
|---|---|
| `auto` |当嵌入模型回答时使用向量，否则使用全文 |
| `lexical` |仅全文 |
| `hybrid` |需要载体；回答`503 hybrid_unavailable`而不是静静地后退|

平局由 `uri` 和 `id` 打破，因此对相同数据的相同查询返回相同的顺序。

> **注：矢量比例**
>
> 向量存储在 SQLite 中并在 Engine 进程中进行比较，没有近似的最近邻居索引。
> 矢量搜索时间随着块的数量而增长，因此随着工作空间的增长，请观察 [metrics](/docs/optimal-engine/operations#observability) 中的 `search_latency_ms`。

## 询问

```bash
curl -s -X POST http://127.0.0.1:4200/api/rag \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What did we decide about pricing?", "workspace": "default:notes", "format": "claude", "bandwidth": "small"}'
```

引擎读取问题的意图，首先查找精选的 wiki 页面，在没有页面时进行搜索，然后将结果适合读者。

|领域|价值观 |
|---|---|
| `format` | `plain`、`markdown`、`claude`、`openai` |
| `bandwidth` | `small`、`medium`、`large` |
| `strategy` | `tiered`、`reconstructive`、`hybrid` |
| `audience` |谁会阅读它，用于过滤和塑造结果 |
| `context_package` | `true` 也返回受控上下文包 |

```json
{
  "source": "chunks",
  "envelope": {
    "body": "...",
    "format": "claude",
    "sources": ["optimal://..."],
    "warnings": []
  },
  "trace": {"intent": "...", "n_candidates": 12, "n_delivered": 4, "elapsed_ms": 41}
}
```

`source` 表示上下文的来源：`wiki`、`chunks`、`memory`、`context_package` 或 `empty`。
`GET /api/rag/stream` 返回与服务器发送事件相同的步骤：`intent`、`wiki_hit` 或 `chunks`、`composing`、`envelope` 和 `done`。

## 组装和重建

`POST /api/assemble` 返回智能体在执行操作之前应阅读的内容：层 `l0` 到 `l3`、`total_tokens`、`sources` 和 `context_package`。
上下文包仅保存经过审查的事实和内存对象，因此在其声明被提升后，新的源会出现在那里。

`POST /api/reconstruct` 使用重构策略：它扩展了读者可以看到的记忆之间的关联。
联想可以提高记忆力，但它们永远无法决定什么是真实的。
使用 `POST /api/reconstruct/:id/outcome` 记录重建的有用性，引擎根据工作空间和意图将其归功于此。

## 审查索赔

```bash
optimal claims -w default:notes
optimal review -w default:notes
```|路线 |它有什么作用 |
|---|---|
| `GET /api/memory-core/claim-review` |审核队列 |
| `GET /api/memory-core/claims` |索赔及其状态和来源|
| `POST /api/memory-core/claims/:id/promote` |接受声明作为事实 (`claims:review`) |
| `POST /api/memory-core/claims/:id/reject` |拒绝索赔 (`claims:review`) |

当分数达到工作区阈值 0.85 时，默认规则会自行提升声明。
设置 `OPTIMAL_CLAIM_PROMOTION_MODE=manual` 自行审核每项声明。

## 版本化记忆

```bash
curl -s -X POST http://127.0.0.1:4200/api/memory/remember \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace": "default:notes", "content": "Customer Example Co renews in March."}'
```

`remember` 首先对记忆进行评分，并用 `action` 回答：

| `action` |意义|
|---|---|
| `add` |存储为新内存 |
| `update` |存储为类似内存的新版本|
| `skip` |已经知道；没有任何改变|
| `pending_claim` |提议作为索赔进行审查 |
| `rejected` |大门下方|

`gate` 对象解释分数：`novelty`、`salience`、`prediction_error` 和 `reason`。
`POST /api/memory` 存储没有门的存储器。

|路线 |它有什么作用 |
|---|---|
| `GET /api/memory` |列表，带有 `q`、`include_forgotten` 和分页 |
| `GET /api/memory/:id/versions` |版本链|
| `GET /api/memory/:id/relations` |链接：`updates`、`extends`、`derives`、`contradicts`、`cites` |
| `POST /api/memory/:id/update`、`extend`、`derive` |新的链接版本 |
| `POST /api/memory/:id/forget` |隐藏它；行保持|
| `DELETE /api/memory/:id` |永久删除 |

## 时间旅行

`search`、`grep`、`ask` 和 `assemble` 拍摄 `as_of`，一个 ISO 8601 瞬间。
事实具有其有效期，而被取代的事实仍像历史一样具有可读性。

## 基准测试

在 LoCoMo 上，所有 1,540 个问题中，引擎的候选组合为 96.1% 的问题和前 100 名所有黄金证据的 87.1% 检索到至少一项黄金证据。
这衡量的是检索率，而不是答案的准确性。
候选投资组合是选择加入的（内存搜索中的`memory_search_mode=portfolio`）；默认内存搜索为 `hybrid`。
存储库的自述文件包含完整的结果、它们的限制以及在 `benchmarks/` 下重现它们的命令。
