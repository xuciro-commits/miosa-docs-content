---
slug: "projects/members-and-usage"
title: "Members and usage"
description: "Project members, permissions, usage attribution, the operating checklist, and common mistakes."
section: "Services & projects"
order: 1003
---

Access follows the hierarchy and narrows at the project:

- Every Workspace member inherits access to every project in that workspace.
- A **project member** row narrows one user's role for one project. Roles are `owner`, `admin`, `developer`, and `viewer`, and the user must already belong to the parent workspace.
- Project endpoints are gated by the `projects:read` and `projects:write` scopes. An API key bound to a workspace pins `workspace_id` for you, so you do not repeat it in the body.
- Member management lives at `GET` and `POST /api/v1/projects/{id}/members` and `PATCH` and `DELETE /api/v1/projects/{id}/members/{user_id}`.

Usage is attributed by the `project_id` on every metered row.
The project roll-up is available at `GET /api/v1/projects/{id}/usage`, and `GET /api/v1/usage/projects` groups settled cost and runtime hours by project or by workspace.
Usage that carries no project is reported in an explicit `unattributed` bucket rather than merged into a named project, so a total that is short is visibly short.
Your `external_project_id` flows through to usage, audit, and list filters.

## Operating checklist

- Every resource resolves to exactly one Organization and Workspace.
- Every product resource has the intended `project_id`.
- Development happens in a Sandbox, not the live Deployment.
- Production changes only through publish, verify, and promote.
- Data survives Sandbox replacement and application rollback.
- The UI displays the authoritative URL returned by the API.
- The project is not the Default project if you plan to delete or move it.

## Common mistakes

| Symptom | Likely cause | Correction |
|---|---|---|
| Resources appear under the wrong customer | Workspace or Project was inferred from stale UI state | Send the canonical IDs explicitly |
| A new app overwrites an older app's history | Both products share one Project and Deployment | Create a separate Project and Deployment |
| A domain change does not affect Sandbox previews | A production hostname was changed | Configure the Project preview domain separately |
| Deleting a Sandbox appears to delete the app | Development and production were treated as one resource | Publish to a Deployment before removing development state |
| A delete returns `409 PROJECT_HAS_RESOURCES` | Sandboxes, deployments, or data still carry the `project_id` | Move or delete the listed resources first, then delete |
| A delete or move returns `409 DEFAULT_PROJECT_PROTECTED` | The target is the workspace's Default project | Create a named project and move the resources into it |

## Next

- [Projects API](/docs/api-reference/projects): Create, resolve, update, and inspect Projects.
- [Workspaces](/docs/api-reference/workspaces): The team, customer, or business boundary above the project.
- [Usage and billing](/docs/platform/usage-and-billing): Per-customer usage and chargeback on a single MIOSA bill.
