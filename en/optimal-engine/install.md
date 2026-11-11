---
slug: "optimal-engine/install"
title: "Install and configure Optimal Engine"
description: "Requirements, the three ways to install a self-hosted Optimal Engine (checkout, Docker, release), every setting it reads, where it stores data, embeddings, and how API keys and scopes work."
section: "Optimal Engine"
order: 8003
---

This page covers a self-hosted Engine.
A managed Engine on MIOSA is set up for you; see [Optimal Engine on MIOSA](/docs/optimal-engine/miosa).

## Requirements

| Need | Version or note |
|---|---|
| Erlang/OTP | 27 or newer. Avoid OTP 28.0 to 28.2; the Docker image uses 28.5. |
| Elixir | 1.17 or newer |
| Build tools | git, a C toolchain, CMake. The image also installs zlib, zstd, lz4, bz2, snappy, and SQLite headers. |
| macOS | `brew install snappy` |
| Embeddings (optional) | [Ollama](https://ollama.com/) with `nomic-embed-text`, or any OpenAI-compatible embeddings endpoint |
| Media parsing (optional) | `pdftotext` for PDF, `tesseract` for image text, `ffmpeg` for video, a whisper.cpp server for audio |

Without an embedding model the Engine still works: search is full-text only, and a backfill embeds everything once a model is set up.
Without a media tool, a file of that kind is stored with its metadata and a warning instead of its text.

## Install

### From a checkout

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
mix deps.get
mix optimal.auth mint --name "root" --workspace '*'
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true mix run --no-halt
```

`make bootstrap` (or `mix optimal.bootstrap`) compiles, runs migrations, and loads the bundled `sample-workspace/` so you have data to try.

### With Docker

`deploy/Dockerfile.engine` builds an OTP release on Elixir 1.19.5 and OTP 28.5.

| Image default | Value |
|---|---|
| User | `optimal`, UID 1000, not root |
| Data | `/data`, the only volume. The database is `/data/.optimal/index.db`. |
| Port | `4200`, bound to `0.0.0.0` inside the container |
| API | On, with keys required (`OPTIMAL_AUTH_REQUIRED=true`) |
| Embeddings | `OLLAMA_HOST=http://host.docker.internal:11434` |
| Health check | `GET /api/health` with `Authorization: Bearer $OPTIMAL_ENGINE_API_KEY` |

Set `OPTIMAL_ENGINE_API_KEY` on the container, or the health check runs without a key and the container reports unhealthy.
On Linux, `host.docker.internal` does not resolve by default; point `OLLAMA_HOST` at the host's address.

### With Docker Compose

```bash
cd deploy
cp env.example .env
# Set OPTIMAL_BOOTSTRAP_KEY and OPTIMAL_ENGINE_API_KEY to the same oe_ key.
docker compose up --build
```

Compose publishes the Engine on `127.0.0.1:4200` and keeps data in the `optimal_data` volume.
For a public server, add the production overlay, which puts Caddy with automatic TLS in front and keeps the Engine port private:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

Set `DOMAIN` and `LETSENCRYPT_EMAIL` in `.env` first.

> **Caution: The storage overlay**
>
> `deploy/docker-compose.storage.yml` also starts Postgres, NATS, Valkey, Qdrant, and OpenBao.
> In 0.3.10 the Engine only uses its S3-compatible storage from that set.
> The others have no adapter yet and report `adapter_missing`.

### As a release

```bash
MIX_ENV=prod mix release optimal
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true _build/prod/rel/optimal/bin/optimal start
```

`bin/optimal maintenance <task>` runs maintenance with the API off; see [Operations](/docs/optimal-engine/operations#upgrades).

## Settings

The Engine reads its settings from environment variables at boot.
An invalid value stops the boot with a message that names the variable.

### Server and keys

| Variable | Default | What it does |
|---|---|---|
| `OPTIMAL_API_ENABLED` | off (image: on) | Only the value `true` starts the HTTP API |
| `OPTIMAL_API_PORT` | `4200` | Port |
| `OPTIMAL_API_INTERFACE` | `127.0.0.1` (image: `0.0.0.0`) | Address to bind |
| `OPTIMAL_AUTH_REQUIRED` | `false` (image: `true`) | Require an API key on every request. Takes `true` or `false` only. |
| `OPTIMAL_BOOTSTRAP_KEY` | none | Registers a first key at boot, in the form `oe_<24 hex>_<secret of 32+ characters>`. Changing it rotates the key. |
| `OPTIMAL_BOOTSTRAP_SCOPES` | `read,write` | Scopes of the bootstrap key |
| `OPTIMAL_BOOTSTRAP_WORKSPACES` | `*` | Workspaces of the bootstrap key |
| `OPTIMAL_BOOTSTRAP_TENANT` | `default` | Tenant of the bootstrap key |
| `OPTIMAL_REFUSE_OPEN_NETWORK` | off | Refuse to start when bound beyond loopback with keys off |
| `OPTIMAL_OPEN_NETWORK_ACKNOWLEDGED` | off | You run keys-off behind a proxy that authenticates every request |
| `OPTIMAL_SIZE_PROFILE` | `business` | Size profile; see [Operations](/docs/optimal-engine/operations#sizing) |

### Paths and storage

| Variable | Default | What it does |
|---|---|---|
| `OPTIMAL_ENGINE_ROOT` | the working directory (image: `/data`) | Root of the workspace folders |
| `OPTIMAL_ENGINE_DB` | `.optimal/index.db` | SQLite database file |
| `OPTIMAL_ENGINE_CACHE` | `.optimal/cache` | Cache, including stored embeddings |
| `OPTIMAL_BACKUP_DIR` | `backups/` next to the database | Where backups are written |
| `OPTIMAL_KNOWLEDGE_BACKEND` | `ets` | Graph projection: `ets` in memory, or `rocksdb` on disk |
| `OPTIMAL_ASSET_STORAGE` | filesystem | `s3` keeps uploaded file bytes in S3-compatible storage |
| `OPTIMAL_S3_ENDPOINT`, `OPTIMAL_S3_BUCKET`, `OPTIMAL_S3_ACCESS_KEY_ID`, `OPTIMAL_S3_SECRET_ACCESS_KEY` | none | S3, MinIO, R2, or Garage. Also `OPTIMAL_S3_REGION` (`us-east-1`) and `OPTIMAL_S3_PREFIX`. |

### Secrets

Each is 64 hex characters; generate one with `openssl rand -hex 32`.

| Variable | Needed when |
|---|---|
| `CONNECTOR_KEY` | You set up app connectors or replication peers. It encrypts their stored keys. |
| `OPTIMAL_AUTHORITY_KEY` | Recommended always. It protects the signing keys of authority grants. |
| `OPTIMAL_BACKUP_KEY` | You send backups offsite. It encrypts them. |

### Embeddings

| Variable | Default | What it does |
|---|---|---|
| `OLLAMA_HOST` | `http://localhost:11434` | Ollama server |
| `OPTIMAL_EMBED_MODEL` | `nomic-embed-text` | Embedding model, 768 dimensions |
| `OPTIMAL_EMBED_URL` | none | Another endpoint. A path ending in `/api/embed` speaks Ollama; anything else speaks OpenAI `/v1/embeddings`. |
| `OPTIMAL_EMBED_DIMENSIONS` | `768` | Vector size of that model |
| `OPTIMAL_EMBED_API_KEY` or `OPTIMAL_EMBED_API_KEY_FILE` | none | Key for that endpoint, read per request and never stored |
| `OPTIMAL_GENERATE_MODEL` | `qwen3:1.7b` | Small model used to curate wiki pages and summarize sessions |
| `OPTIMAL_VLM_MODEL` | `qwen2.5-vl` | Vision model that describes images |

### Behavior

| Variable | Default | What it does |
|---|---|---|
| `OPTIMAL_CLAIM_PROMOTION_MODE` | `auto_threshold` | `auto_threshold`, `manual`, or `agent_review`; see [Concepts](/docs/optimal-engine/concepts#claims-and-facts) |
| `OPTIMAL_AUTHORITY_MODE` | `off` | `off`, `observe` (log decisions), or `enforce` |
| `OPTIMAL_ENGINE_SCOPE` | `org` | `workspace` limits the Engine to `OPTIMAL_ENGINE_WORKSPACE` |
| `OPTIMAL_IMPORT_MAX_BYTES` | 1 GiB | Largest import file |
| `OPTIMAL_BUNDLE_MAX_BYTES` | 5 GiB | Largest workspace bundle |
| `OPTIMAL_RELEASE_CHANNEL` | `dev` | `dev`, `stable`, `beta`, or `pinned`, as `/api/version` reports it |

`deploy/env.example` in the repository lists every variable with a comment.

## Where data lives

SQLite is the store.
It holds full-text search, vectors, and every record, so one file plus the folders around it is the whole Engine.

```text
<OPTIMAL_ENGINE_ROOT>/          workspace folders: nodes/, .wiki/, assets/
.optimal/index.db               the database (with -wal and -shm files)
.optimal/cache/                 cache and stored embeddings
.optimal/backups/               local backups
.optimal/knowledge-rocksdb/     graph projection, when the rocksdb backend is on
```

`GET /api/stores` lists the 13 logical stores and which provider backs each one.

> **Note: PostgreSQL**
>
> A PostgreSQL store exists, but it needs a build compiled with `OPTIMAL_STORE=postgres`, and moving an existing Engine onto it is not supported yet.
> The Docker image is SQLite only.
> Run `mix optimal.postgres.check` to see its state on your build.

## API keys and scopes

Send a key as `Authorization: Bearer <key>` or `X-API-Key: <key>`.
Keys look like `oe_<id>_<secret>`; the Engine keeps only a bcrypt hash of the secret.

Mint keys from the checkout:

```bash
mix optimal.auth mint --name "agent" --workspace default:notes --scope read --scope write
mix optimal.auth list
mix optimal.auth revoke <id>
```

Or over HTTP with an `admin` key: `POST /api/auth/keys` with `{"name", "scopes", "workspace_scope", "expires_at"}`.
A key can only mint keys narrower than itself.
`GET /api/auth/whoami` shows a key's own scopes and workspaces.

| Scope | Allows |
|---|---|
| `read`, `write` | Reading and writing in the key's workspaces |
| `admin` | Keys, backups, maintenance, and other operator routes |
| `claims:review` | Promoting and rejecting claims |
| `topology:write` | Changing workspaces and nodes |
| `connectors:write` | Setting up app connectors |
| `sync` | Replication |
| `data:admin` | App data collections |
| `authority:read`, `authority:enforce`, `authority:approve`, `authority:issue`, `authority:admin` | Authority rules and grants |
| `*` | Everything |

> **Caution: Keys off means open**
>
> With `OPTIMAL_AUTH_REQUIRED=false`, a request with no key acts as the local operator and can do anything.
> That is fine on `127.0.0.1`.
> Bound to any other address, the Engine logs an error at boot and every 10 minutes, and `/api/health` warns `auth_open_on_network`.
> A tunnel or proxy in front of a loopback Engine is not detected, so turn keys on before you publish it.
