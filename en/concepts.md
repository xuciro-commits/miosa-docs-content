---
slug: "concepts"
title: "Core Concepts"
description: "The canonical MIOSA product model, from mutable sandboxes to immutable releases, stable deployments, persistent data, and customer-owned infrastructure."
section: "Optional"
order: 17005
---

MIOSA gives developers and agents a safe development loop without making production mutable.
You work in a Sandbox, inspect the result through a Preview, and publish a frozen candidate Release.
After MIOSA verifies that candidate, promotion moves the Deployment's stable URL to the new Version.

If you read only one line, remember this:

**Sandbox (mutable) → Preview → Publish → Release (immutable) → Verify → Promote → Deployment (stable URL).**

The old Version remains available for rollback.
Databases, storage, secrets, and domains remain attached to the Deployment instead of being copied into each release.

> **Note: The central boundary**
>
> Changing a Sandbox changes development state.
> It does not change production.
> Production changes only when an exact immutable candidate is promoted.

This page is the **resource model**: what the MIOSA resources are and how they relate.
For the **API protocol** - authentication, token types, idempotency, rate limits, errors, and events - see [API fundamentals](/docs/fundamentals).

## Pick what you need

| You want to | Read |
|---|---|
| The ownership chain and the development and production resources | [Resource model](/docs/concepts/resource-model) |
| How a publish becomes a live release | [The publish pipeline](/docs/concepts/publish-pipeline) |
| Databases, storage, and bindings | [Data and bindings](/docs/concepts/data-and-bindings) |
| The desktop product | [Computers](/docs/concepts/computers-desktop) |
| White-label tenancy and attribution | [Tenancy](/docs/concepts/tenancy) |
| Browser tokens for previews and shares | [Preview and share tokens](/docs/concepts/tokens) |
| Tenant preview domain and branding | [Tenant configuration](/docs/concepts/tenant-config) |

New to the platform vocabulary?
Keep the [MIOSA Glossary](/docs/glossary) open while you follow the resource and publishing flows below.

## Next

- [Resource model](/docs/concepts/resource-model): The ownership chain, Sandboxes, and Previews - the mutable side of the platform.
- [The publish pipeline](/docs/concepts/publish-pipeline): Builder, Release, Runtime Instance, Domain, Deployment, and Version: how source becomes a live release.
- [Data and bindings](/docs/concepts/data-and-bindings): Postgres, Redis, object storage, volumes, auth, and how they attach to a Deployment.
- [Computers](/docs/concepts/computers-desktop): The cloud desktop product, and the difference between a Computer and a Sandbox.
- [Tenancy](/docs/concepts/tenancy): White-label tenancy and external attribution for platforms built on MIOSA.
- [Tokens](/docs/concepts/tokens): Preview tokens and share tokens for browser access to a sandbox preview.
- [Tenant configuration](/docs/concepts/tenant-config): Tenant preview domain and tenant branding.

## Deep dives

**Computers (desktop product)** now lives in [Computers](/docs/concepts/computers-desktop).

**Tenant preview domain** now lives in [Tenant configuration](/docs/concepts/tenant-config).
