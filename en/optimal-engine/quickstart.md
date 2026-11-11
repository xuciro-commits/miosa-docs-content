---
slug: "optimal-engine/quickstart"
title: "Optimal Engine quickstart"
description: "Run Optimal Engine on your own machine, mint an API key, create a workspace, store a source, and search it - from a checkout or with Docker - plus the shortest path on MIOSA."
section: "Optimal Engine"
order: 8002
---

This guide runs one Engine on your machine, creates a workspace, stores a source, and finds it again.
It takes about ten minutes, most of it the first compile.
To use a managed Engine on MIOSA instead, skip to [On MIOSA](/docs/optimal-engine/quickstart#on-miosa).

You end with:

- an Engine on `http://127.0.0.1:4200`;
- an API key that scopes every request;
- a workspace `default:notes` with one stored source;
- a search result that points back to that source.

## 1. Run the Engine

**From a checkout**

You need Erlang/OTP 27 or newer, Elixir 1.17 or newer, git, and a C toolchain.
On macOS, also run `brew install snappy`.

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
mix deps.get
```

Mint the first key before the server starts.
The key is printed once, so copy it now.

```bash
mix optimal.auth mint --name "root" --workspace '*'
```

Start the Engine with the HTTP API on and keys required:

```bash
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true mix run --no-halt
```

The first start runs every database migration before the API answers.

**With Docker**

You need Docker.
The first image build compiles RocksDB and can take 30 to 60 minutes.

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
docker build -f deploy/Dockerfile.engine \
  --build-arg OPTIMAL_RELEASE_CHANNEL=stable \
  -t optimal-engine:quickstart .
```

Mint the first key in a one-off container that opens the same volume with the API off:

```bash
docker run --rm -v optimal-data:/data -e OPTIMAL_API_ENABLED=false optimal-engine:quickstart \
  /app/bin/optimal eval \
  'OptimalEngine.Auth.ApiKey.mint(%{tenant_id: "default", name: "root", scopes: ["*"], workspace_scope: ["*"]}) |> IO.inspect()'
```

Start the Engine.
`OPTIMAL_ENGINE_API_KEY` is the key you just minted; the container health check uses it.

```bash
docker run -d --name optimal-engine -p 127.0.0.1:4200:4200 \
  -v optimal-data:/data \
  -e OPTIMAL_AUTHORITY_KEY="$(openssl rand -hex 32)" \
  -e CONNECTOR_KEY="$(openssl rand -hex 32)" \
  -e OPTIMAL_ENGINE_API_KEY="<your key>" \
  optimal-engine:quickstart
```

The image turns on the API and requires keys by default.
Everything the Engine stores lives in the `optimal-data` volume, so replacing the container keeps your data.

## 2. Check that it answers

In a second terminal:

```bash
export OPTIMAL_ENGINE_API_KEY="<your key>"
curl -s http://127.0.0.1:4200/api/health -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

```json
{
  "status": "up",
  "live": true,
  "checks": {
    "store": ":ok",
    "migrations": ":ok",
    "credential_key": ":ok",
    "network_auth": ":ok"
  },
  "degraded": [],
  "warnings": []
}
```

`status` is `up`, `degraded` when there is a warning, or `down` when a check fails.
The real response also carries the release identity.

## 3. Create a workspace

```bash
curl -s -X POST http://127.0.0.1:4200/api/workspaces \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"slug": "notes", "name": "Notes"}'
```

The response is the new workspace.
Its `id` is `default:notes`; use that id everywhere below.

## 4. Store a source

```bash
curl -s -X POST http://127.0.0.1:4200/api/ingest \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "title": "Supplier renewal call",
    "text": "Renewal call with the supplier moved to the 14th. The contract review is still open."
  }'
```

```json
{
  "ok": true,
  "signal_id": "...",
  "genre": "...",
  "type": "...",
  "entities": [],
  "source_package_id": "..."
}
```

The Engine kept the source, classified it, and proposed claims from it.
A claim becomes a fact once it is reviewed or meets the workspace's promotion rule; see [Concepts](/docs/optimal-engine/concepts#claims-and-facts).

## 5. Search and ask

```bash
curl -s "http://127.0.0.1:4200/api/search?q=supplier%20renewal&workspace=default:notes&limit=5" \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

Each result has a `title`, the matching `content`, a `score`, and a `uri` in the `optimal://` scheme that points back to the source.

```bash
curl -s -X POST http://127.0.0.1:4200/api/rag \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What is open with the supplier?", "workspace": "default:notes", "format": "markdown"}'
```

`envelope.body` is the context an answer should be written from, and `envelope.sources` lists the sources it used.
The Engine does not write the answer itself; hand the envelope to your model or agent.

> **Tip: Vector search is optional**
>
> Out of the box, search is full-text.
> Run [Ollama](https://ollama.com/) with `nomic-embed-text`, or point `OPTIMAL_EMBED_URL` at an embeddings endpoint, to add vector search.
> See [Install and configure](/docs/optimal-engine/install#embeddings).

## 6. Use the client CLI

The repository ships a small client CLI, `optimal` (also called `oe`), that needs only `sh`, `curl`, and `awk`.

```bash
apps/cli/install.sh
export OPTIMAL_ENGINE_URL=http://127.0.0.1:4200
optimal doctor
optimal search "supplier renewal" -w default:notes
```

See [CLI, SDKs and MCP](/docs/optimal-engine/cli-sdks-mcp) for every command and for connecting an agent over MCP.

## On MIOSA

Managed Engines are enabled per organization.
Once yours is enabled:

1. Open **Optimal > Memory** in the console and pick a workspace.
2. Press **Start Memory**. The console shows the size and the hourly price before it creates anything.
3. When Memory is running, search it on the same page, or ask Optimal in a chat to save a note to Memory.

From a terminal, the [MIOSA CLI](/docs/cli) does the same:

```bash
miosa engine create notes-brain --workspace notes --for workspace --wait
miosa engine connect notes-brain
optimal search "supplier renewal"
```

See [Optimal Engine on MIOSA](/docs/optimal-engine/miosa) for what is included and how to ask for access.
