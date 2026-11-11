---
slug: "support"
title: "Support"
description: "How to reach the MIOSA team, what to include so a ticket resolves fast, and where to check status and what has shipped."
section: "Help"
order: 16012
---

## Get in touch

- **In-product ticket.** Open a support ticket from the MIOSA dashboard. This is the fastest path for account-specific issues because it carries your tenant context.
- **Email.** Write to `platform@miosa.ai` for anything that is not tied to a live resource, such as roadmap questions or a limitation you need lifted.

## What to include

A precise report resolves in one round trip. Include:

- The **resource id** (sandbox, computer, deployment, secret) and your tenant or workspace.
- The **operation** that failed and how you invoked it (SDK call, CLI command, raw request).
- A **timestamp with timezone**, and the request's **`x-request-id`** header if you have it.
- The **error text** exactly as it came back, including the status code.
- What you already checked, such as the pending network queue or build logs.

> **Caution: Redact secrets before you send**
>
> Remove API keys, tokens, host keys, private URLs that contain tokens, and any sensitive command output. Never paste a live `msk_*` key or a secret value into a ticket.

## Before you open a ticket

Many issues have a documented fix:

- [Troubleshooting](/docs/troubleshooting) is the front door for sandbox, exec, preview, network, and deploy problems.
- [Security troubleshooting](/docs/security/troubleshooting) covers secrets, OAuth, the network allowlist, and the audit log.
- `miosa doctor` checks your CLI version, config, endpoint, key, and clock skew in one command.

## Status and changes

- **Status page.** The status page shows current platform health and recent incidents. Check it first if something looks broadly broken rather than scoped to your tenant.
- **Changelog.** The [changelog](/docs/changelog) lists what shipped, and [Known limitations](/docs/known-limitations) lists what has not.

## Raising a limit

Concurrency, rate limits, and volume terms can be raised for a plan. Ask through the same channels and include your expected scale; see [Pricing and limits](/docs/pricing) for the current defaults.

- [Troubleshooting](/docs/troubleshooting): Common failures and where to look for each.
- [Known limitations](/docs/known-limitations): What does not work yet, updated every release.
- [Status](https://miosa.ai/status): Current platform health and incident history.
