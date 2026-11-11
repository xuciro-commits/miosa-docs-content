---
slug: "security/troubleshooting"
title: "Security  -  Troubleshooting"
description: "Top 15 issues across MIOSA Secrets, Connect Accounts, Network Allowlist, Audit Log, and White-label Credentials - with root causes and fixes."
section: "Security & compliance"
order: 14005
---

Quick-reference for the 15 most common issues across all MIOSA Security features. Each entry: symptom → root cause → fix.

---

## Secrets

### 1. Secret value is not being swapped: outbound request uses the raw placeholder

**Symptom**: Your sandbox sees `miosa-tok-...` in `$OPENAI_API_KEY` and that placeholder reaches the upstream API, which returns a 401.

**Root cause**: The proxy swaps placeholders by scanning the outbound request for known token prefixes. If the request uses a non-standard header name or doesn't include the `Authorization` header at all, the swap is skipped.

**Fix**:
- Confirm the SDK or library you're using puts the token in the `Authorization: Bearer ...` header.
- For query-string auth (e.g., `?api_key=...`), make sure the token appears verbatim in the query string  -  the proxy scans query params too.
- Check the audit row's `request_transforms.secrets.swapped` field. If it's empty, the proxy didn't match the placeholder.
- Verify the env var name casing matches exactly. `OPENAI_API_KEY` ≠ `openai_api_key`. The proxy is case-sensitive when looking up `expose_as_env` bindings.

---

### 2. `secrets.list()` shows the secret but `$ENV_VAR` is empty inside the sandbox

**Symptom**: The secret is listed in the dashboard and SDK, but `echo $OPENAI_API_KEY` inside the sandbox returns nothing.

**Root cause**: The secret was created with `scope="tenant"` or `scope="workspace"` but the sandbox was not created under the same workspace. Or the `expose_as_env` value has a typo.

**Fix**:
```python
# Check the binding on the sandbox directly
secrets = sandbox.secrets.list()
for s in secrets:
    print(s.name, s.expose_as_env, s.scope)

# If missing, bind it explicitly to this sandbox
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

---

### 3. Rotating a key doesn't take effect  -  sandbox still uses the old key

**Symptom**: You called `sandbox.secrets.rotate(name, new_value=...)` but outbound calls to the API still succeed with what appears to be the old key.

**Root cause**: This is almost always correct behavior. Rotation is instant  -  the proxy looks up the new value on the next request carrying the placeholder. If the upstream call succeeds after rotation, the new key is already working. The old key may still be valid on the provider's side (keys aren't invalidated on MIOSA's end  -  only on the provider's end).

**Fix**: Verify rotation worked by checking the audit log:
```python
events = sandbox.audit.list(since="5m", host="api.openai.com", limit=5)
for e in events:
    print(e.timestamp, e.status_code, e.request_transforms)
```
If `status_code` is 200 after rotation, the new key is in use.

To force the old key to stop working: revoke it on the provider's side (e.g., delete it in the OpenAI dashboard), then rotate on MIOSA.

---

## Connect Accounts (OAuth)

### 4. GitHub OAuth refresh failing  -  sandbox gets 401 on `api.github.com`

**Symptom**: The sandbox was working, then suddenly starts getting 401s from GitHub. The audit log shows `action=allow` but `status_code=401`.

**Root cause**: The GitHub access token expired and the refresh attempt failed. This happens when:
- The user revoked the GitHub OAuth app's access from their GitHub settings
- The refresh token itself expired (GitHub personal OAuth tokens don't expire, but GitHub Apps tokens do after 8 hours without a refresh)
- The scopes were changed on the provider side

**Fix**:
1. In the MIOSA dashboard → Secrets tab, the GitHub entry will show "Reconnect needed."
2. Call `flow.reauthorize()` to re-run the OAuth dance:
```python
flow = sandbox.secrets.reauthorize(provider="github")
print("Authorize at:", flow.authorize_url)
flow.wait_for_completion(timeout=300)
```

---

### 5. Custom OAuth provider returns "invalid_state"

**Symptom**: The OAuth callback from a custom OAuth2 provider includes `error=invalid_state`.

**Root cause**: The `state` parameter in the OAuth dance has a 10-minute TTL. If the user took more than 10 minutes to authorize, or if the callback was called twice (e.g., the user hit the back button and tried again), the state is expired or consumed.

**Fix**: Re-initiate the flow. State is one-time use.
```python
# Don't reuse the old flow object  -  create a new one
flow = sandbox.secrets.connect(
    provider="my_custom_provider",
    expose_as_env="MY_API_TOKEN",
)
print("New authorize URL:", flow.authorize_url)
```
If you're building the UI, show the user a "Try again" button that calls your backend to initiate a fresh `connect()`.

---

### 6. Provider OAuth app shows "Authorize MIOSA" instead of my brand

**Symptom**: Your end-users see the MIOSA branding on the GitHub (or other provider) consent screen.

**Root cause**: You haven't registered a custom OAuth app override for your tenant.

**Fix**: See [White-label OAuth setup](/docs/security/white-label#white-label-oauth-your-branding-not-miosas). Register your own OAuth app with the provider, then add the override in Settings → OAuth Providers.

---

## Network Allowlist

### 7. Sandbox can't reach `pypi.org` after lockdown

**Symptom**: `pip install <package>` fails with a connection error after you locked down the network.

**Root cause**: `pip` contacts three domains, and all three must be on the allowlist:

| Domain | What it does |
|---|---|
| `pypi.org` | Package index queries |
| `files.pythonhosted.org` | Package download CDN |
| `pythonhosted.org` | Legacy package downloads (some packages still use this) |

**Fix**:
```python
sandbox.network.allow("pypi.org")
sandbox.network.allow("files.pythonhosted.org")
sandbox.network.allow("pythonhosted.org")
# If you also need conda:
sandbox.network.allow("conda.anaconda.org")
sandbox.network.allow("repo.anaconda.com")
```

---

### 8. `npm install` fails after lockdown

**Symptom**: `npm install` or `pnpm install` fails after switching to enforce mode.

**Root cause**: The npm registry and its CDN are not on the allowlist.

**Fix**:
```python
sandbox.network.allow("registry.npmjs.org")
sandbox.network.allow("*.npmjs.com")
```
If you use pnpm's own mirror:
```python
sandbox.network.allow("registry.npmjs.org")
```

---

### 9. Agent stopped calling a host that's on the allowlist

**Symptom**: You can see the host in your allowlist, but the agent keeps getting blocked.

**Root cause**: The agent is resolving a subdomain or alias not covered by your rule. For example, `api.openai.com` is on the list but the agent is also hitting `oai.endpoint.amazonaws.com` (OpenAI's backend CDN).

**Fix**:
1. Check the Pending Requests queue:
```python
pending = sandbox.network.pending()
for p in pending:
    print(p.host, p.request_count)
```
2. Add any missing hosts to the allowlist.
3. For CDNs and dynamic subdomains, use a wildcard: `*.openai.com`.

---

### 10. I locked down a sandbox but all new sandboxes from the same template are locked too

**Symptom**: Sandboxes created from a template inherit enforce mode unexpectedly.

**Root cause**: The template you snapshotted was already in enforce mode. The policy state is part of the template snapshot.

**Fix**: After creating a sandbox from the template, immediately call:
```python
new_sandbox.network.observe()
```
To prevent this for all future sandboxes, snapshot the template while it's in audit-only mode.

---

## Audit Log

### 11. Audit log is empty  -  no rows for a sandbox that's been running

**Symptom**: `sandbox.audit.list(since="24h")` returns an empty list, even though the sandbox is running and making calls.

**Root cause**: Either:
- The sandbox isn't making any *outbound* calls (the audit log only records egress, not inbound traffic)
- The sandbox was created before the MIOSA egress feature was enabled on your tenant

**Fix**: Confirm outbound traffic from inside the sandbox:
```python
r = sandbox.exec("curl -s -o /dev/null -w '%{http_code}' https://api.openai.com/")
print(r.stdout)  # 401 is fine  -  it proves egress happened
```
Then re-query the audit log. If still empty after confirmed egress, file a support ticket.

---

### 12. Audit rows show `request_transforms.secrets.swapped = []` even though a secret is bound

**Symptom**: The request to `api.openai.com` succeeds, but the audit row shows no secrets were swapped.

**Root cause**: The secret binding was added *after* this particular request was made, or the sandbox resolved the real key from a `.env` file rather than the placeholder (so no swap was needed).

**Fix**: Check whether the env var contains a placeholder or the real key:
```python
r = sandbox.exec("echo $OPENAI_API_KEY")
print(r.stdout)
# "miosa-tok-..." means placeholder is present → proxy will swap
# "sk-proj-..."  means real key is in the env → proxy has nothing to swap
```
If the real key is in the env var (e.g., from a `.env` file), the proxy never sees the placeholder and no swap occurs.

---

## White-label

### 13. Audit query by `external_user_id` returns rows from other users

**Symptom**: `client.audit.list(external_user_id="alice@example.com")` returns events that clearly belong to other users.

**Root cause**: This should not happen  -  `external_user_id` is a strict equality filter within your tenant. Most commonly this is a wildcard match being applied in your own code.

**Fix**: Verify you're passing the exact ID, not a pattern:
```python
# Correct
events = client.audit.list(external_user_id="alice@example.com")

# Wrong  -  this is not a prefix match, but double-check your exact string
events = client.audit.list(external_user_id="alice@")  # won't match anything
```
If the issue persists, check that multiple sandboxes aren't all being created with the same `external_user_id` by mistake.

---

### 14. `scope="external_user"` secret returns 403

**Symptom**: Calling `client.secrets.set(..., scope="external_user", external_user_id="...")` returns HTTP 403.

**Root cause**: Your tenant has "Allow end-user-scoped credentials" disabled.

**Fix**: In the MIOSA dashboard → Settings → Security → toggle "Allow end-user-scoped credentials" to On. If you intentionally disabled it and this call is from your own backend (not an end-user), use `scope="tenant"` or `scope="external_workspace"` instead.

---

### 15. Resolver scope walk skips an expected secret

**Symptom**: A sandbox with `external_user_id="alice"` doesn't pick up a secret set at `scope="external_workspace"` for `external_workspace_id="team-a"`, even though Alice is in team-a.

**Root cause**: The resolver scope walk uses the `external_workspace_id` stamped on the sandbox at creation time. If the sandbox was created without `external_workspace_id="team-a"`, the workspace-level secret doesn't resolve for it.

**Fix**: Verify the sandbox has the right workspace stamp:
```python
sbx = client.sandboxes.get(sandbox_id)
print(sbx.external_workspace_id)   # should be "team-a"
```
If it's missing or wrong, create a new sandbox with the correct attribution. Attribution fields are immutable after creation.

---

## Still stuck?

- Use the Audit Log → detail drawer: each row shows exactly which transforms ran and which rules fired.
- Check [Network Allowlist](/docs/security/network), [Secrets](/docs/security/secrets), [Connect Accounts](/docs/security/connect-accounts) for full reference docs.
- For bugs, open a support ticket from the MIOSA dashboard  -  include the sandbox ID and the relevant audit row ID.
