---
slug: "services/domains"
title: "Services addresses and domains"
description: "How services reach databases while private networking is not available, the public miosa.app address a web service or static site can have, and how TLS is handled."
section: "Networking & domains"
order: 5000
---

Services reach managed databases through their connection variables.
A web service or a static site can also have a public address.

## Private networking

Private networking between services is not available yet.
One service cannot reach another by a private name, and `private_network` reads `{"enabled": false}`.
The older `private_domain` field (`<service-slug>.miosa.internal`) and the `MIOSA_PRIVATE_DOMAIN` variable do not resolve; do not build on them.

Managed databases connect automatically from any host.
Reference a database's connection in the service's Variables:

```text
DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}
```

To call another service of the project, give it a public address and call that.
The service's Networking tab in the console shows the same.

## Public address

```bash
miosa service domains shop API
miosa service domains enable shop API --label api
miosa service domains disable shop API
```

`enable` gives the service:

```text
https://<label>.<your-org>.miosa.app
```

The label defaults to the service's slug and must be lowercase a-z, 0-9 and `-`.
The label is unique across your organization.
TLS is issued and renewed by the platform; there is no certificate to install.

`disable` removes the public address and leaves the private one in place.
The service keeps running; it just stops being reachable from the internet.

## What serves a public address

The public address routes to the healthy replicas of the service's active release.
A release that is not `active` does not receive traffic.

If the service has idle sleep enabled, a request to a sleeping service wakes it and waits for it.
A service without a public address has no request signal, so it never sleeps.

## Reading the address in code

TypeScript:

```typescript
const service = await miosa.services.get(serviceId);
console.log(service.private_network);       // { enabled: false } while private networking is not available
console.log(service.public_address.url);    // null until enabled
```

Python:

```python
service = miosa.services.get(service_id)
print(service["private_network"])
print(service["public_address"]["url"])
```

## Custom domains

The public address is on `miosa.app`.
A custom domain on your own DNS zone is a separate product: register it, add the CNAME it shows and let MIOSA verify it.
That flow lives under Domains, not under a service, so one domain can outlive a service.

## When ingress is off

If public addresses are not switched on for the platform, `enable` answers `INGRESS_DISABLED` (404).
The service still runs, but has no address to reach it on, and `miosa service settings shop` reports `public_address` as `unsupported`.
