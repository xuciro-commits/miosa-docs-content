---
slug: "optimal-engine"
title: "Optimal Engine"
description: "Optimal Engine is the self-hosted memory and knowledge engine behind a workspace - it keeps source evidence, builds reviewed memory, and returns cited context to people and agents over HTTP, a CLI, and MCP."
section: "Optimal Engine"
order: 8000
---

Optimal Engine is a memory and knowledge engine for a person, a team, or a company.
It keeps the sources you give it, turns them into memory that someone has reviewed, and returns cited context to people and agents.
It is open source under the Apache 2.0 license, at [github.com/Miosa-osa/OptimalEngine](https://github.com/Miosa-osa/OptimalEngine).

Optimal Engine is not the Optimal chat agent.
[Optimal](/docs/optimal) is the agent you chat with in the console.
Optimal Engine is the store it remembers with: a server you run yourself, or a managed one on MIOSA.

## What it does

| Step | What the Engine does |
|---|---|
| Organize | Keeps a tree of organizations, workspaces, and typed nodes such as people, products, and projects |
| Capture | Stores raw text, files, chat sessions, Notion exports, and records pulled from apps such as Slack, GitHub, and Linear |
| Remember | Classifies each input, extracts claims, and promotes a claim to a fact only after review or a confidence rule |
| Retrieve | Searches with full-text ranking, adds vector search when an embedding model is set up, and returns context with its sources |
| Operate | Gives agents a daily brief, open loops, a protocol, and allow, ask, or deny rules for what they may do |
| Project | Writes the same state out as markdown files, wiki pages, workspace bundles, and an HTTP API |

The Engine does not write answers with a language model.
`ask` returns the context and the sources an answer should come from, and your agent or model writes the answer.

## Two ways to run it

| | Self-hosted | On MIOSA |
|---|---|---|
| Where it runs | Your machine, server, or container | A private machine in your MIOSA workspace |
| Who runs it | You: install, keys, backups, upgrades | MIOSA: provisioning, backups, upgrades, health checks |
| Search | Full-text, plus vector search with your own embedding model | Full-text |
| How you reach it | `http://127.0.0.1:4200` or your own domain | The **Memory** page, Optimal chat, the `miosa engine` commands, and machines in the workspace |
| Availability | Anyone, from the repository | Enabled per organization |

Self-hosting needs nothing from MIOSA.
See the [self-hosted quickstart](/docs/optimal-engine/quickstart) or [Optimal Engine on MIOSA](/docs/optimal-engine/miosa).

> **Note: Current release**
>
> These pages describe Optimal Engine 0.3.10, API version `v1`.
> Check what a running Engine reports with `GET /api/version`.

## Where to go next

- [Concepts](/docs/optimal-engine/concepts): Workspaces, nodes, sources, claims, facts, and context packages.
- [Quickstart](/docs/optimal-engine/quickstart): Run an Engine, store a source, and search it in about ten minutes.
- [Install and configure](/docs/optimal-engine/install): Requirements, Docker, settings, storage, and API keys.
- [Memory and retrieval](/docs/optimal-engine/memory-and-retrieval): How search, ask, and the review of claims work.
- [API reference](/docs/optimal-engine/api): Authentication, errors, and the main routes with examples.
- [Optimal Engine on MIOSA](/docs/optimal-engine/miosa): Workspace Memory, managed Engines, and keys for the CLI.
