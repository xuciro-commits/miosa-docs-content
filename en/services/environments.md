---
slug: "services/environments"
title: "Services environments"
description: "Every project starts with a production environment. Fork it into staging or a preview, and switch between environments on the canvas."
section: "Services & projects"
order: 1010
---

An environment is a copy of a project that runs on its own.
Every project starts with a `production` environment.
Add `staging`, a preview, or anything else you need.

Each environment has its own services, variables and releases.
A change in staging does not touch production.

## Switch and fork on the canvas

The environment name sits in the canvas top bar, next to the project name.
Open it to see every environment of the project and to switch between them.
The canvas reloads for the environment you pick, and the address carries it as `?env=<name>`.

Choose **Fork** to create a new environment from the one you are looking at.
Give it a name.
MIOSA copies the services and variables, then shows a summary of what was copied and what was skipped.

## From the API

```text
GET  /api/v1/projects/:id/environments
POST /api/v1/projects/:id/environments/:env/duplicate   {"name": "staging"}
```

`duplicate` returns the new environment, the services and variables it copied, and the entries it skipped.
The project's service routes accept `?env=<name>`; without it they use `production`.

## Where to go next

- [Variables](/docs/services/variables): variables and references are per environment.
- [Releases and rollback](/docs/services/releases): each environment has its own releases.
- Docker Host has its own [environments](/docs/docker-host/environments) for the former Docker Host deployments.
