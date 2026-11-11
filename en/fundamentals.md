---
slug: "fundamentals"
title: "Fundamental Concepts"
description: "The mental model every MIOSA integrator needs - workspace, authentication, resources, attribution, idempotency, rate limits, errors, and events."
section: "API reference"
order: 12001
---

MIOSA is a cloud platform for building and deploying applications. Before writing your first API call, read this page. It covers the concepts that underpin every MIOSA integration.

This page is the **API protocol** reference: authentication, tokens, idempotency, rate limits, errors, events, and tenant-level configuration.
For the **resource model** - what the resources are and how they relate - see [Core Concepts](/docs/concepts).

## What every integration needs

| Topic | Read |
|---|---|
| API keys, browser tokens, and the five token types | [Authentication](/docs/fundamentals/authentication) |
| Mapping resources to your own customers | [Ownership and external attribution](/docs/fundamentals/attribution) |
| Safe retries, quotas, and error handling | [Requests, limits, and errors](/docs/fundamentals/requests) |
| Streaming progress over SSE | [Events (SSE)](/docs/fundamentals/events) |
| Webhooks and per-end-user quotas | [Tenant configuration](/docs/fundamentals/tenant-config) |

## Where resources live

This page covers the API protocol. The full **resource model** - the ownership chain, the development and production resources, and how they relate - lives in [Core Concepts](/docs/concepts).

In short: your account is one **organization** (a permanent slug, one billing account, and API keys). Inside it, **workspaces** separate customers, clients, teams, or environments, and **projects** group the resources for one product. Sandboxes, Computers, Deployments, Databases, Domains, Storage, Volumes, Functions, and Jobs belong to a project. Every request resolves one unambiguous organization, workspace, and project.

## Sandboxes vs Computers

Sandboxes and Computers are two different products with different lifecycles and different use cases. The comparison table and the resource-model context live in [Core Concepts](/docs/concepts#computers-desktop-product).

- Use a **Sandbox** when the agent needs to write code, run shell commands, manage files, or serve a dev server.
- Use a **Computer** when the agent needs to operate a graphical interface - a browser, a desktop app, or anything with a visible screen.

Both products live in the same organization, can be separated by workspace/project, and appear on the same billing account.

**Ownership and external attribution** now lives in [Ownership and external attribution](/docs/fundamentals/attribution).

## Next

- [Authentication](/docs/fundamentals/authentication): API key formats, browser tokens, and the five token types with their trust levels.
- [Ownership and external attribution](/docs/fundamentals/attribution): Canonical ownership fields, external attribution, and usage rollup for platforms.
- [Requests, limits, and errors](/docs/fundamentals/requests): Idempotency keys, rate limits, and the error envelope with HTTP status codes.
- [Events (SSE)](/docs/fundamentals/events): Stream progress with one-time tickets, resource streams, and the tenant-wide stream.
- [Tenant configuration](/docs/fundamentals/tenant-config): Webhook subscriptions and per-end-user quotas for white-label integrators.

- [Quickstart](/docs/quickstart): Five-minute hands-on tour - create a sandbox, write code, publish to a live URL.
- [Sandboxes](/docs/develop/sandboxes): Full sandbox lifecycle - create, exec, files, snapshots, auto-suspend.
- [Templates](/docs/develop/templates): Pre-built images for Node, Next.js, Python, Go, Rust, and more.
- [Deploy Overview](/docs/docker-host/overview): Builder, Release, Version, Runtime Instance - the full publish pipeline.
