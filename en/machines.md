---
slug: "machines"
title: "Machine capabilities"
description: "What every machine ships with, its sizes, runtimes, and pre-installed tools."
section: "Sandboxes"
order: 2013
---

A machine is the compute behind a sandbox or a computer: a versioned template, a named size, a runtime engine, and the tools already installed in the image.
Pick the closest template to your stack, choose a size, and MIOSA boots the machine ready to run your code.

## Templates

A **template** is a prebuilt sandbox image with a specific language, framework, or toolchain already installed.
Pick the closest match to your stack; you can override install/start commands per sandbox if needed.
See [Templates](/docs/develop/templates) for the built-in images and the custom BuildSpec reference.

## Sizes

Every sandbox runs on a named resource contract.
The default is `small`: 2 vCPU, 4096 MiB RAM, and 10240 MiB disk.
Prefer a named `size` over raw CPU, memory, and disk fields.
See [Sizing and limits](/docs/develop/sandboxes/sizing) for the contracts and what each costs.

## Runtimes

A sandbox runs on one of two engines: **Firecracker**, MIOSA's long-standing default, or **SOMA**, MIOSA's own open-source VMM.
Your organization chooses which runtimes it offers, and a sandbox records the runtime it was created with.
See [Choosing a runtime](/docs/soma/choosing-a-runtime) to list the runtimes you can pick and what each reports it supports today.

## Pre-installed tools

Templates ship with their toolchains already in the image, so there is nothing to install before your first command.
Docker with the `docker compose` and `docker buildx` plugins is available in every sandbox, including SOMA runtimes.

## Next

- [Templates](/docs/develop/templates): The built-in images and what each is prebuilt for, plus custom BuildSpecs.
- [Sizing and limits](/docs/develop/sandboxes/sizing): Named resource contracts, raw overrides, and billing notes.
- [Choosing a runtime](/docs/soma/choosing-a-runtime): Firecracker, SOMA, or auto, and how to list what your organization offers.
- [Sandboxes](/docs/develop/sandboxes): The full sandbox product: exec, files, previews, snapshots, and lifecycle.
