---
slug: "faq/getting-started"
title: "Getting started"
description: "Where to start, how to authenticate, install the CLI, get an API key, attach IDs to a sandbox, and pick a region."
section: "Help"
order: 16001
---

Short answers on the first steps with MIOSA.

Open a category, or use Expand all and Collapse all above the list.
Every question has its own link - the icon on the right - so you can share an answer directly.

## Where do I start?

The [Quickstart](/docs/quickstart) recommends **Track 1: sandbox to deployment** first.
Your first success is a site live at the production URL MIOSA returns, verified end to end.
The full walkthrough is [Build and deploy a Next.js app](/docs/learn/first-production-app).

## How do I authenticate?

With a MIOSA API key in an `Authorization: Bearer` header, or with a JWT from `POST /auth/login`.
See [Authentication](/docs/authentication) and [API keys](/docs/platform/api-keys).

## How do I install the CLI and sign in?

Install the `miosa` CLI (v2.2) from npm, Homebrew, the install script, or a release archive.
Then run `miosa login` for the device-code flow and `miosa whoami` to confirm the active organization and workspace.
See [Install the CLI](/docs/cli/install).

```bash
miosa login     # device-code flow
miosa whoami    # confirm the active organization and workspace
```

## How do I get an API key?

Create one under **Workspace → Settings → API Keys**.
The plaintext is shown once and only the hash is stored.
Key families are `msk_u_` (user), `msk_p_` (tenant platform), and `msk_a_` (operator admin, not self-service).
Default rates are 600 requests per minute for a user key and 30,000 for a platform key; rotate every 90 days.
See [API keys](/docs/platform/api-keys).

## Is there a test mode?

No.
There are no dedicated test-mode keys: `msk_u_*` and `msk_p_*` keys hit the billable production substrate.
Use a sandbox or a separate workspace for experiments.

## What IDs can I attach to a sandbox?

Free-form `metadata`, plus the queryable `external_user_id`, `external_workspace_id`, and `external_project_id`.
See [Metadata and labels](/docs/develop/sandboxes/metadata).

## Which regions exist, and can a sandbox move between them?

`us-west` (default), `us-east`, and `us-mia`, all generally available.
A resource lives in one region for its lifetime, so create one per region.
EU and APAC are on the roadmap.
See [Regions](/docs/regions).
