---
slug: "services/protection"
title: "Protection and WAF"
description: "Put a password in front of a service, mint share links, and set the web application firewall rules the proxy enforces."
section: "Networking & domains"
order: 5005
---

Protection and the WAF are enforced at the edge, in front of the service, so they
apply before a request reaches your process.

## Password and share links

```bash
miosa protection shop API                            # is it protected?
miosa protection shop API --password 'open-sesame'
miosa protection shop API --share-link --expires 3600
miosa protection shop API --revoke <link-id>
miosa protection shop API --off
```

The password is never shown again. A share link is a way past it for one person:
the `msl_...` token is shown once, and expires after `--expires` seconds (seven
days by default, at most ninety). A visitor uses it as `?miosa_share=<token>` or
with an `x-miosa-share-token` header.

Protection can also be set on a Deploy target or on an environment kind, and the
rows resolve service over environment over kind. That is why a preview environment
can be protected without touching each service in it.

## WAF

```bash
miosa waf shop API
miosa waf shop API --mode enforce --deny 203.0.113.0/24 --rate 120
miosa waf shop API --allow 198.51.100.0/24
miosa waf shop API --attack-mode on
miosa waf shop API --off
```

- `--mode` is `off`, `observe` or `enforce`.
- `--allow`/`--deny` take IPs or CIDRs (`ip_allow`/`ip_deny` on the wire).
- `--rate` is requests per minute per source; `0` clears the limit.
- `--attack-mode` serves a JavaScript challenge to browsers and `403` to
  everything else.

**Country rules read only the trusted `x-miosa-country` header that the edge
sets.** There is no GeoIP lookup of our own: if a request arrives without that
header the country rules are skipped, and the WAF document says whether they can
act at all with `country_rules_active`.

> **Note: Observe before enforce**
>
> Run `--mode observe` for a day first. The WAF reports what it would have blocked,
> so you can see a wrong CIDR before it turns into a page full of 403s.
