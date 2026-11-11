---
slug: "data/public-endpoint"
title: "Public database endpoint"
description: "Open a TLS-only TCP endpoint for one managed database, with an allowlist, so a laptop or an outside service can connect."
section: "Networking & domains"
order: 5006
---

A managed database is reachable from inside the platform by its private URL. Turn
on public networking when something outside needs to connect: a laptop, a
migration tool, an analytics job in another cloud.

```bash
miosa db public app-db                       # off or on, and the address
miosa db public app-db on                    # any source
miosa db public app-db on --allow 203.0.113.0/24 --allow 198.51.100.7
miosa db public app-db off                  # close it and free the port
```

The same calls are `GET`, `POST` and `DELETE /api/v1/databases/:id/public-networking`.

## What you get

- A host and a port from **30000-39999**, one port per database. The host is the
  edge's public address, so it does not change when the database moves host.
- `public_url` in the credentials response while it is on, and
  `DATABASE_PUBLIC_URL` in the database service's variables, so
  `${{db.DATABASE_PUBLIC_URL}}` resolves for anything that needs it.

## Security

- **TLS only.** Postgres needs `sslmode=require`; MySQL needs
  `--ssl-mode=REQUIRED` and refuses a plaintext login with error 3159. The
  certificate is self-signed, so `require` encrypts without verifying the name.
- **The allowlist closes disallowed sources before any TLS handshake.** `--allow`
  replaces the stored list, so `on` with no `--allow` means any source. Entries
  are IPs or CIDRs, at most 32.
- Limits: 100 connections per database, 20 new connections per 10 seconds per
  source IP, at most 32 TLS handshakes at once, and 2,000 public connections
  across your databases.

## Engines

Postgres, MySQL, Redis and MongoDB. Other engines answer `ENGINE_NOT_SUPPORTED`.

## Turning it off

`off`, or deleting the database, closes the port within about fifteen seconds.
Both are recorded in the audit log as `database.public_networking_enable` and
`database.public_networking_disable`.

> **Note: Prefer the private URL**
>
> Anything inside the platform should use the private URL. The public endpoint
> exists for clients that cannot reach the private network, not as the default way
> to connect.
