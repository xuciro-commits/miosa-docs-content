---
slug: "data-retention"
title: "Data retention and deletion"
description: "What MIOSA keeps, for how long, and what \"delete\" actually removes for sandboxes, snapshots, the audit log, usage records, and stored data."
section: "Security & compliance"
order: 14003
---

This page describes what MIOSA keeps and for how long, and what each delete operation removes. Where a retention window is not published, the page says so rather than guessing.

## Sandboxes

Deleting a sandbox (`DELETE /api/v1/sandboxes/{id}`) permanently deletes the record, removes its saved filesystem state, and settles billing. There is no undo. Use pause when the user is done for now but may come back.

A paused sandbox keeps its workspace until it is destroyed. Destroy it when the saved state should be removed.

## Snapshots

Snapshots have their own lifecycle and can end in a `deleted` state, either because you deleted them or because a retention policy removed them. Deleting a snapshot does not affect sandboxes that were previously restored from it. See [Snapshots and persistence](/docs/develop/snapshots) for the full status list.

If you are relying on a specific retention window for snapshots, confirm it for your tenant before you depend on it.

## Audit log

Egress audit events are retained for **90 days** for every tenant, and longer on enterprise plans. The [Egress API](/docs/api-reference/egress) documents the audit query endpoint and its retention.

The audit log is for security and compliance, not billing. Usage records are separate and carry the same attribution.

## Usage and credit records

Usage records drive billing and chargeback. Credit balances and transactions are read through the [Credits API](/docs/api-reference/credits); credits expire **180 days** from purchase.

## Webhook events

Failed webhook deliveries are retried eight times over roughly **15.5 hours** (within a 24-hour window). After that the event is dropped, so if you need to backfill missed events, read the audit log or usage records instead of relying on redelivery.

## Volumes and stored data

Sandboxes and storage services are separate lifecycles. Destroying a sandbox does not delete a [Volume](/docs/data/volumes) or an object storage bucket; delete those explicitly when their data should go.

## Stricter retention

MIOSA does not publish a zero-data-retention mode. If you need stricter or shorter retention than the defaults here, contact [support](/docs/support) to discuss the terms for your tenant before you build on them.

- [Snapshots and persistence](/docs/develop/snapshots): What survives pause, restore, and fork, and what a snapshot does not save.
- [Audit log](/docs/security/audit-log): What every audit row records and how to query it.
- [Support](/docs/support): How to reach the team for retention and compliance questions.
