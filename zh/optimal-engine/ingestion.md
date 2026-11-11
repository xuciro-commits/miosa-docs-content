---
slug: "optimal-engine/ingestion"
title: "来源和摄入"
description: "将数据导入 Optimal Engine 的各种方式 - 文本、文件、聊天会话、Notion 导出、批量导入以及 Slack、GitHub 和 Linear 等应用程序 - 以及它解析的文件类型和适用的限制。"
section: "Optimal Engine"
order: 8005
---

每个输入首先保存为源包，然后分类、分割成块、索引并挖掘声明。
原件永远不会被重写，因此每个事实都可以追溯到它的来源。

## 方法

|输入 |路线或命令 |
|---|---|
|文字| `POST /api/ingest`、`optimal capture`、`mix optimal.ingest` |
|一个文件 | `POST /api/assets` 字节为 base64 |
|文件的文件夹 | `mix optimal.ingest_workspace` |
|聊天完毕 | `POST /api/sessions/commit`、`optimal session commit` |
|持久的记忆 | `POST /api/memory`、`POST /api/memory/remember`、`optimal remember` |
|概念导出 | `POST /api/import/notion`、`mix optimal.import_notion` |
|一次处理多个记录 | `POST /api/batch/import/signals`、`POST /api/batch/import/memories` |
|一个应用程序 |连接器，如下|
|另一个引擎的工作区 |工作区捆绑包；请参阅[操作](/docs/optimal-engine/operations#move-a-workspace) |

## 文本

```bash
curl -s -X POST http://127.0.0.1:4200/api/ingest \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "title": "Pricing decision",
    "text": "Decided: annual plans get two months free from March.",
    "node": "pricing",
    "extract_claims": true
  }'
```

|领域|必填 |意义|
|---|---|---|
| `text` |是的 |内容 |
| `workspace` |没有|工作区 ID； `default` 省略时 |
| `title` |没有|结果中显示的标题 |
| `genre` |没有|覆盖引擎选择的类型 |
| `node` |没有|源所属的节点 slug |
| `extract_claims` |没有|默认情况下为 `true` |

响应是`{"ok": true, "signal_id", "genre", "type", "entities", "source_package_id"}`。
缺少的 `text` 回答 `400`；管道无法使用 `ok: false` 和 `error` 获取答案 `422`。

## 文件

`POST /api/assets` 将文件作为 `content_base64` 及其文件名。
引擎将散列后的字节存储在工作区的 `assets/` 文件夹下，或者在 `OPTIMAL_ASSET_STORAGE=s3` 时存储在 S3 兼容存储中。

|亲切 |扩展 |文字来自 |
|---|---|---|
| Markdown 和文本 | `.md` `.txt` `.rst` `.adoc` `.log` |文件 |
|数据| `.yaml` `.yml` `.toml` `.json` `.csv` `.tsv` |文件 |
|网页 | `.html` `.htm` `.xhtml` |文件 |
|代码|约 37 种语言，例如 `.ex` `.py` `.js` `.ts` `.go` `.rs` `.java` `.sql` |文件 |
|办公室 | `.docx` `.pptx` `.xlsx` |文件 |
| PDF | `.pdf` | `pdftotext` |
|图片 | `.png` `.jpg` `.jpeg` `.gif` `.webp` `.svg` | `tesseract`，加上视觉模型的描述 |
|音频| `.mp3` `.wav` `.m4a` `.ogg` `.flac` |一个耳语.cpp 服务器 |
|视频 | `.mp4` `.mov` `.webm` `.mkv` | `ffmpeg` 关键帧和音频 |

未知的扩展名被读取为纯文本。
当工具丢失时，文件将与其元数据和警告（例如 `ffmpeg not on PATH` 一起保留，您可以稍后再次解析它。

## 聊天会话

提交完成的对话，以便将其教训变成记忆：

```bash
curl -s -X POST http://127.0.0.1:4200/api/sessions/commit \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "correlation_id": "chat-2026-10-23-01",
    "title": "Renewal planning",
    "messages": [
      {"role": "user", "content": "When is the supplier renewal?"},
      {"role": "assistant", "content": "The call moved to the 14th."}
    ],
    "learned": ["The supplier renewal call is on the 14th."]
  }'
```

聊天内容被存储为一个片段。
每个 `learned` 项目（最多 50 个）将成为审核队列中的提议索赔。
响应 (`201`) 具有 `session_id`、其 `uri`、`summary` 和 `message_count`。
使用 `GET /api/sessions` 和 `GET /api/sessions/:id` 回读会话。

## 概念导出

将 Notion 工作区导出为 **Markdown 和 CSV**，然后发送 zip：

```bash
mix optimal.import_notion ./notion-export.zip --workspace default:notes
```

通过 HTTP，`POST /api/import/notion` 将 zip 作为请求正文并启动作业；后面跟 `GET /api/import/notion/:job_id`。
每个页面、数据库和行都成为具有 `optimal://<workspace>/notion/<id>` URI 的自己的项目，页面层次结构和链接成为图形边缘。
导入程序在解压任何内容之前检查大小，拒绝 ZIP64 存档，并且从不将条目名称转换为磁盘上的路径。

## 应用程序（连接器）

连接器按计划从应用程序中提取记录并将其存储为源。
它只会读取；没有任何内容写回应用程序。

|集团|应用程序 |
|---|---|
|邮件、文件和文档 | Gmail、Google Drive、Notion、Dropbox、OneDrive、Confluence、本地源文件夹 |
|聊天和会议 | Slack、Microsoft Teams、Zoom、Fathom |
|工作跟踪和代码 | GitHub、线性、Jira |
| CRM 与业务 | HubSpot、Salesforce、HighLevel、Stripe |
|出版和表格| Beehiiv，Jotform |

在添加之前设置 `CONNECTOR_KEY`：它会对引擎存储的应用程序密钥进行加密。

```bash
mix optimal.connector list
mix optimal.pull
```通过 HTTP，`GET` 和 `POST /api/connectors` 列出并添加连接器，并且 `POST /api/connectors/:id/run` 现在运行一个。
存储库中的 `docs/guides/connectors.md` 列出了每个人提取的内容以及登录方式。
提取的记录将存储为具有 `optimal://connectors/<kind>/<id>` URI 的数据库行，而不是文件，除非连接器设置为写入文件。

> **注意：连接器的深度**
>
> 每个连接器都会拉取其指南列表中的记录。
> 尚未完成每​​个应用程序的每个附件和二进制文件的下载。

## 限制

|限制|价值|
|---|---|
| JSON 请求正文 | 8MB |
|导入文件 | 1 GiB（`OPTIMAL_IMPORT_MAX_BYTES`）|
|工作区捆绑包 | 5 GiB（`OPTIMAL_BUNDLE_MAX_BYTES`）|
|发送到嵌入模型的文本 |每块 6,000 个字符 |
|每次聊天会话学习的项目 | 50 |
