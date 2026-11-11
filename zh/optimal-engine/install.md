---
slug: "optimal-engine/install"
title: "安装和配置Optimal Engine"
description: "要求、安装自托管Optimal Engine的三种方法（签出、Docker、发布）、它读取的每个设置、存储数据的位置、嵌入以及 API 密钥和范围如何工作。"
section: "Optimal Engine"
order: 8003
---

本页介绍了一个自托管引擎。
MIOSA 上的托管引擎已为您设置；请参阅[MIOSA 上的Optimal Engine](/docs/optimal-engine/miosa)。

## 要求

|需要|版本或注释 |
|---|---|
| Erlang/OTP | 27 或更新。避免 OTP 28.0 至 28.2； Docker 镜像使用 28.5。 |
| Elixir | 1.17 或更高版本 |
|构建工具| git，一个 C 工具链，CMake。该映像还安装 zlib、zstd、lz4、bz2、snappy 和 SQLite 标头。 |
| macOS | `brew install snappy` |
|嵌入（可选）| [Ollama](https://ollama.com/) 与 `nomic-embed-text` 或任何 OpenAI 兼容的嵌入端点 |
|媒体解析（可选）| `pdftotext` 用于 PDF，`tesseract` 用于图像文本，`ffmpeg` 用于视频，用于音频的 whisper.cpp 服务器 |

如果没有嵌入模型，引擎仍然可以工作：搜索仅是全文，并且一旦模型设置好，回填就会嵌入所有内容。
如果没有媒体工具，此类文件将与其元数据和警告一起存储，而不是其文本。

## 安装

### 从结账处

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
mix deps.get
mix optimal.auth mint --name "root" --workspace '*'
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true mix run --no-halt
```

`make bootstrap`（或 `mix optimal.bootstrap`）编译、运行迁移并加载捆绑的 `sample-workspace/`，以便您可以尝试数据。

### 使用 Docker

`deploy/Dockerfile.engine` 在 Elixir 1.19.5 和 OTP 28.5 上构建 OTP 版本。

|镜像默认值 |值 |
|---|---|
|用户| `optimal`，UID 1000，不是 root |
|数据| `/data`，唯一的卷。数据库是`/data/.optimal/index.db`。 |
|港口| `4200`，绑定到容器内的 `0.0.0.0` |
| API |开启，且必须提供密钥（`OPTIMAL_AUTH_REQUIRED=true`）|
|嵌入 | `OLLAMA_HOST=http://host.docker.internal:11434` |
|健康检查| `GET /api/health` 与 `Authorization: Bearer $OPTIMAL_ENGINE_API_KEY` |

在容器上设置 `OPTIMAL_ENGINE_API_KEY`，或者在没有密钥的情况下运行运行状况检查，并且容器报告运行状况不佳。
在 Linux 上，`host.docker.internal` 默认情况下不解析；将 `OLLAMA_HOST` 指向主机地址。

### 使用 Docker 撰写

```bash
cd deploy
cp env.example .env
# Set OPTIMAL_BOOTSTRAP_KEY and OPTIMAL_ENGINE_API_KEY to the same oe_ key.
docker compose up --build
```

Compose 在 `127.0.0.1:4200` 上发布引擎并将数据保存在 `optimal_data` 卷中。
对于公共服务器，添加生产覆盖层，它将带有自动 TLS 的 Caddy 放在前面，并保持引擎端口的私有性：

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

首先在`.env`中设置`DOMAIN`和`LETSENCRYPT_EMAIL`。

> **注意：存储覆盖**
>
> `deploy/docker-compose.storage.yml` 还启动 Postgres、NATS、Valkey、Qdrant 和 OpenBao。
> 在 0.3.10 中，引擎仅使用该组中的 S3 兼容存储。
> 其他人还没有适配器并报告`adapter_missing`。

### 作为一个版本

```bash
MIX_ENV=prod mix release optimal
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true _build/prod/rel/optimal/bin/optimal start
```

`bin/optimal maintenance <task>` 在 API 关闭的情况下运行维护；请参阅[操作](/docs/optimal-engine/operations#upgrades)。

## 设置

引擎在启动时从环境变量中读取其设置。
无效值会停止启动，并显示一条命名该变量的消息。

### 服务器和密钥

|变量|默认 |它有什么作用 |
|---|---|---|
| `OPTIMAL_API_ENABLED` |关闭（镜像中：开启）|只有 `true` 才启动 HTTP API |
| `OPTIMAL_API_PORT` | `4200` |端口 |
| `OPTIMAL_API_INTERFACE` | `127.0.0.1`（镜像中：`0.0.0.0`）|绑定的地址 |
| `OPTIMAL_AUTH_REQUIRED` | `false`（镜像中：`true`）|每个请求都必须提供 API 密钥。只接受 `true` 或 `false`。 |
| `OPTIMAL_BOOTSTRAP_KEY` |无 |在启动时注册第一个键，格式为 `oe_<24 hex>_<secret of 32+ characters>`。更改它会旋转密钥。 |
| `OPTIMAL_BOOTSTRAP_SCOPES` | `read,write` |引导键的范围 |
| `OPTIMAL_BOOTSTRAP_WORKSPACES` | `*` |引导键的工作区 |
| `OPTIMAL_BOOTSTRAP_TENANT` | `default` |引导密钥的租户 |
| `OPTIMAL_REFUSE_OPEN_NETWORK` |关闭 |在密钥关闭时绑定到回环以外的地址则拒绝启动 |
| `OPTIMAL_OPEN_NETWORK_ACKNOWLEDGED` |关闭 |您在会认证每个请求的代理后面以关闭密钥的方式运行 |
| `OPTIMAL_SIZE_PROFILE` | `business` |规格档位；请参阅[运维](/docs/optimal-engine/operations#sizing) |

### 路径和存储

|变量 |默认 |它有什么作用 |
|---|---|---|
| `OPTIMAL_ENGINE_ROOT` |工作目录（图像：`/data`）|工作区文件夹的根目录 |
| `OPTIMAL_ENGINE_DB` | `.optimal/index.db` | SQLite 数据库文件 |
| `OPTIMAL_ENGINE_CACHE` | `.optimal/cache` |缓存，包括存储的嵌入 |
| `OPTIMAL_BACKUP_DIR` |数据库旁边的`backups/` |备份写入的位置 |
| `OPTIMAL_KNOWLEDGE_BACKEND` | `ets` |图投影：内存中的 `ets`，或磁盘上的 `rocksdb` |
| `OPTIMAL_ASSET_STORAGE` |文件系统 | `s3` 将上传的文件字节保存在兼容 S3 的存储中 |
| `OPTIMAL_S3_ENDPOINT`、`OPTIMAL_S3_BUCKET`、`OPTIMAL_S3_ACCESS_KEY_ID`、`OPTIMAL_S3_SECRET_ACCESS_KEY` |无 | S3、MinIO、R2 或 Garage。还有`OPTIMAL_S3_REGION`（`us-east-1`）和`OPTIMAL_S3_PREFIX`。 |

### 秘密

每个都是 64 个十六进制字符；使用 `openssl rand -hex 32` 生成一个。

|变量|何时需要 |
|---|---|
| `CONNECTOR_KEY` |您设置应用程序连接器或复制对等点。它加密他们存储的密钥。 |
| `OPTIMAL_AUTHORITY_KEY` |一直推荐。它保护授权的签名密钥。 |
| `OPTIMAL_BACKUP_KEY` |您将备份发送到异地。它对它们进行加密。 |

### 嵌入

|变量|默认 |它有什么作用 |
|---|---|---|
| `OLLAMA_HOST` | `http://localhost:11434` | Ollama 服务器 |
| `OPTIMAL_EMBED_MODEL` | `nomic-embed-text` |嵌入模型，768 个维度 |
| `OPTIMAL_EMBED_URL` |无 |另一个端点。以 `/api/embed` 结尾的路径说 Ollama；其他任何东西都讲 OpenAI `/v1/embeddings`。 |
| `OPTIMAL_EMBED_DIMENSIONS` | `768` |该模型的矢量大小 |
| `OPTIMAL_EMBED_API_KEY` 或 `OPTIMAL_EMBED_API_KEY_FILE` |无 |该端点的密钥，按请求读取且从不存储 |
| `OPTIMAL_GENERATE_MODEL` | `qwen3:1.7b` |用于管理 wiki 页面和总结会话的小型模型 |
| `OPTIMAL_VLM_MODEL` | `qwen2.5-vl` |描述图像的视觉模型|

### 行为

|变量|默认 |它有什么作用 |
|---|---|---|
| `OPTIMAL_CLAIM_PROMOTION_MODE` | `auto_threshold` | `auto_threshold`、`manual` 或 `agent_review`；请参阅[概念](/docs/optimal-engine/concepts#claims-and-facts) |
| `OPTIMAL_AUTHORITY_MODE` | `off` | `off`、`observe`（记录决策）或 `enforce` |
| `OPTIMAL_ENGINE_SCOPE` | `org` | `workspace` 将引擎限制为 `OPTIMAL_ENGINE_WORKSPACE` |
| `OPTIMAL_IMPORT_MAX_BYTES` | 1 GiB |最大的导入文件|
| `OPTIMAL_BUNDLE_MAX_BYTES` | 5 GiB |最大的工作空间包 |
| `OPTIMAL_RELEASE_CHANNEL` | `dev` | `dev`、`stable`、`beta` 或 `pinned`，如 `/api/version` 报告 |

存储库中的 `deploy/env.example` 列出了带有注释的每个变量。

## 数据所在的位置

SQLite 是商店。
它包含全文搜索、向量和每条记录，因此一个文件加上其周围的文件夹就是整个引擎。

```text
<OPTIMAL_ENGINE_ROOT>/          workspace folders: nodes/, .wiki/, assets/
.optimal/index.db               the database (with -wal and -shm files)
.optimal/cache/                 cache and stored embeddings
.optimal/backups/               local backups
.optimal/knowledge-rocksdb/     graph projection, when the rocksdb backend is on
```

`GET /api/stores` 列出了 13 个逻辑存储以及哪个提供商支持每个逻辑存储。

> **注意：PostgreSQL**
>
> 存在 PostgreSQL 存储，但它需要使用 `OPTIMAL_STORE=postgres` 编译的版本，并且尚不支持将现有引擎移至其上。
> Docker 镜像仅是 SQLite。
> 运行 `mix optimal.postgres.check` 以查看其在构建中的状态。

## API 密钥和范围

以 `Authorization: Bearer <key>` 或 `X-API-Key: <key>` 形式发送密钥。
键看起来像 `oe_<id>_<secret>`；引擎仅保留秘密的 bcrypt 哈希值。

从结帐处获取密钥：

```bash
mix optimal.auth mint --name "agent" --workspace default:notes --scope read --scope write
mix optimal.auth list
mix optimal.auth revoke <id>
```

或者通过 HTTP 使用 `admin` 键：`POST /api/auth/keys` 和 `{"name", "scopes", "workspace_scope", "expires_at"}`。
密钥只能铸造比其自身窄的密钥。
`GET /api/auth/whoami` 显示键自己的作用域和工作空间。

|范围 |允许 |
|---|---|
| `read`, `write` |在密钥的工作区中读取和写入 |
| `admin` |密钥、备份、维护和其他运维路由 |
| `claims:review` |提升和拒绝声明 |
| `topology:write` |更改工作区和节点 |
| `connectors:write` |设置应用程序连接器 |
| `sync` |复制|
| `data:admin` |应用数据集合 |
| `authority:read`、`authority:enforce`、`authority:approve`、`authority:issue`、`authority:admin` |授权规则和授予 |
| `*` |一切 |

> **注意：密钥关闭意味着开放**
>
> 使用 `OPTIMAL_AUTH_REQUIRED=false`，没有键的请求充当本地操作符并且可以执行任何操作。
> 这在 `127.0.0.1` 上很好。
> 绑定到任何其他地址时，引擎会在启动时和每 10 分钟记录一次错误，并且 `/api/health` 发出警告 `auth_open_on_network`。
> 未检测到环回引擎前面的隧道或智能体，因此在发布之前打开密钥。
