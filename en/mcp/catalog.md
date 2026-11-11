---
slug: "mcp/catalog"
title: "Tool server catalog"
description: "Add an official tool server in one step, test its key before saving, and keep the key in the workspace vault."
section: "CLI, SDKs & MCP"
order: 11031
---

MIOSA keeps a catalog of official tool servers you can add in one step, then attach to your agents.
Open it from **Connections**, then **Tool servers**, then **Browse catalog**.

## What is in the catalog

The catalog lists 40 verified tool servers, including Apify, GitHub, Linear, Notion, Sentry, Stripe, Cloudflare, Firecrawl, Exa, Perplexity, Context7, and Playwright.

Search by name, or filter by category.
Each entry shows what the vendor needs and where to get it.

Some servers only sign in through your browser, for example Figma, Vercel, Google Workspace, and Mobbin.
They are listed, with no key field, and are not available yet.

## Add a key and test it

Adding a server asks for exactly the keys the vendor needs, each with a "Get it here" link.
Before anything is saved, **Test connection** checks the key against the vendor.
The key is sent only to the vendor's own URL, and the test stores nothing.

On success the key is sealed in the workspace vault, and the saved server holds a `{{secret:NAME}}` reference, never the value.
A response never carries a key.

If the name is already taken, you get a clear conflict, and you can add a second account with its own name.

## Which harnesses load tool servers today

A tool server is loaded by the harness whose CLI reads a tool-server config.
Today that is Claude Code, Codex, and OSA.
The generic harnesses, pi, OpenCode, Prime Agent, Kimi Code, and Mistral Vibe, do not load tool servers yet.

## The catalog as an API

| Route | Scope | What it does |
|---|---|---|
| `GET /api/v1/mcp-catalog?workspace_id=` | `connections:read` | Lists entries, categories, and what you already added |
| `POST /api/v1/mcp-catalog/:slug/test` | `connections:write` | Tests a key against the vendor's URL, and stores nothing |
| `POST /api/v1/mcp-catalog/:slug/install` | `connections:write` | Adds the server and seals its keys in the workspace vault |

Install sends `{ workspace_id, method, name, values }` and returns `201` with the server and the secret names, never a value.
A duplicate returns `409 ALREADY_ADDED`.
A bad field returns `422 VALIDATION_ERROR` with details per field.

## Related

- [MCP overview](/docs/mcp): the MIOSA tool server your own agent calls.
- [Harnesses and auth](/docs/agents/harnesses): which agents run on your account.
- [Managed connectors](/docs/guides/managed-connectors): MIOSA-owned tools like Refero.
