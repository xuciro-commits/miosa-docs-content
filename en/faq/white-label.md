---
slug: "faq/white-label"
title: "White-label and platform builders"
description: "White-label architecture, branded URLs, end-user auth, billing your customers, and branding."
section: "Help"
order: 16006
---

Short answers on running MIOSA under your own brand for your customers.

## How does white-label work on MIOSA?

Your platform maps to one MIOSA organization, with a workspace per downstream customer and a project per thing they build.
End users never need a MIOSA account.
See [White-label platform architecture](/docs/platform/overview).

## What branded URLs can I use?

A preview domain (`https://{port}-{slug}.sandbox.{preview_domain}`), a deployment domain (`https://{slug}.{deployment_domain}`), and a MIOSA fallback (`https://{slug}.{organization_slug}.miosa.app`).
See [Branding and domain setup](/docs/platform/white-label).

## How do my end users authenticate?

Never put an `msk_*` key in a browser.
Your backend mints short-lived scoped tokens, and end-user-scoped secrets and OAuth are keyed on `external_user_id`.
See [Attribution](/docs/platform/attribution) and [White-label credentials](/docs/security/white-label).

## How do I bill my customers through MIOSA?

MIOSA charges the organization owner for all usage in one bill.
Charge back to customers by grouping the Usage Rollup on `workspace_id` or the `external_*` fields.
See [Usage and billing](/docs/platform/usage-and-billing).

## Can I brand the console and desktop?

Yes. Set `custom_app_name`, `custom_logo_url`, `support_url`, `desktop_wallpaper_url`, and `powered_by_visible` through `PUT /api/v1/tenant/branding`.
See [Branding and domain setup](/docs/platform/white-label).
