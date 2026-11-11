---
slug: "docker-host/templates"
title: "Docker Host templates"
description: "Curated app shapes for Docker Host - Dockerfile, Compose, command, port, environment, and persistence defaults."
section: "Services & projects"
order: 1022
---

Docker Host templates are curated app shapes for agents, SDKs, and the UI. They do not replace your project code. They provide Dockerfile, Compose, command, port, environment, persistence, and design-context defaults before a sandbox or repository is published.

| Template                      | Best for                                          | Runtime        |
| ----------------------------- | ------------------------------------------------- | -------------- |
| Next.js App                   | SaaS portals, dashboards, client apps             | Node           |
| Next.js Funnel / Landing Page | funnels, lead magnets, campaign sites             | Node           |
| Vite React SPA                | static React apps, calculators, widgets           | Nginx          |
| Express / Node API            | webhooks, integrations, lightweight APIs          | Node           |
| FastAPI Service               | Python APIs, AI helper services, data tools       | Python         |
| Static HTML Site              | documents, one-page tools, static downloads       | Nginx          |
| Compose Full-Stack App        | web + API + worker + Postgres + Redis on one host | Docker Compose |

Template metadata is returned by:

```text
GET /api/v1/docker-deploy/templates
GET /api/v1/docker-deploy/templates/:id
```

Use a template id as metadata when creating a deployment so the deployment center can show the intended build shape and recover the defaults later:

```json
{
  "deployment_product": "docker_deploy",
  "metadata": {
    "docker_deploy_template_id": "compose-full-stack"
  }
}
```

## See also

- [Docker Hosts](/docs/docker-host/hosts): The workspace host that runs Docker Host containers.
- [Publishing](/docs/docker-host/publishing): The publish pipeline that produces a release artifact.
