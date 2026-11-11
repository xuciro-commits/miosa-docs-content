---
slug: "faq/business-os"
title: "Business OS and internal use"
description: "Running your own company on MIOSA, connectors, device choice, and platform checklists."
section: "Help"
order: 16007
---

Short answers on running your own company on MIOSA and building an internal platform.

## Can I run my own company on MIOSA?

Yes. The reference map includes MIOSA BusinessOS, a self-hosted business workspace you run internally rather than launch as a product.
See [Agent company reference map](/docs/guides/agent-company-reference-map).

## How do agents reach my internal tools?

Managed connectors attach in-VM tools without shipping you vendor tokens; the sandbox gets a placeholder swapped only for that provider's endpoint.
See [Managed connectors](/docs/guides/managed-connectors).

## How do I pick a device for each job?

Pure reasoning needs no device; files and code use a Sandbox; browser and desktop work use a Computer; private customer resources use a local device; serving traffic uses Docker Host.
See [Choose an agent device](/docs/guides/agent-device-selection).

## Where do I start building an internal platform?

Follow the build-your-own-platform checklist: workspace, credentials, runtime profile, primary device, dispatch, stream, export, snapshot, publish, scale.
See [Build an agent-company platform](/docs/guides/build-agent-company).
