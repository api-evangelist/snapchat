---
name: snapchat-authenticate
description: >-
  Get and keep a working credential for any Snapchat developer surface — the Marketing API, Login
  Kit "Log in with Snapchat", the Conversions API, or the hosted Ads MCP server. Use when asked to
  authenticate against Snapchat, set up OAuth, connect an agent to Snapchat Ads, or debug a 401.
generated: '2026-08-13'
method: generated
source: >-
  openapi/snapchat-oauth-api-openapi.yml, openapi/snapchat-user-profile-api-openapi.yml,
  authentication/snapchat-authentication.yml, scopes/snapchat-scopes.yml, mcp/snapchat-mcp.yml
api: Snapchat OAuth / Login Kit / Ads MCP
operations:
  - authorize
  - exchangeToken
  - getUserProfile
---

# Authenticate against Snapchat

Everything is OAuth 2.0, but there are **three different authorization servers**. Pick the right
one before you start.

| Surface | Authorization server | What you get |
| --- | --- | --- |
| Marketing API, Login Kit | `accounts.snapchat.com` | Bearer access token, 3600s TTL, refreshable |
| Conversions API | `accounts.snapchat.com` *or* Ads Manager | OAuth token, or a long-lived static token |
| Ads MCP server | `mcp.snapchat.com` | Agent-scoped token, `snapads.read` only |

There is **no OpenID Connect**. `/.well-known/openid-configuration` 404s on every Snap host, no
`id_token` is issued, and identity comes from `GET /me`, not from a claims set.

## Register the app

Create the application and get `client_id` / `client_secret` at
<https://kit.snapchat.com/manage/apps>.

## Flow 1 — Authorization code with PKCE (public clients)

Recommended for mobile apps and SPAs.

`authorize` → `GET https://accounts.snapchat.com/accounts/oauth2/auth`

Required parameters:

| Parameter | Value |
| --- | --- |
| `client_id` | from the developer portal |
| `redirect_uri` | must match the registration exactly |
| `response_type` | `code` |
| `scope` | space-separated, see below |
| `state` | random, CSRF protection |
| `code_challenge` | S256 hash of your `code_verifier` |
| `code_challenge_method` | `S256` |

Then `exchangeToken` → `POST https://accounts.snapchat.com/login/oauth2/access_token` with the
`code` and the original `code_verifier`.

## Flow 2 — Server-side authorization code (confidential clients)

Same endpoints, with `client_secret` instead of PKCE. Gives long-term access through refresh
tokens. Use this for the Marketing API, which is a server-side integration by nature.

## Flow 3 — Implicit grant

`response_type=token` returns the access token straight in the redirect fragment. Snap documents it
for SPAs and says outright it has "limited security compared to other flows". Prefer Flow 1.

## Scopes

Login Kit scopes are fully-qualified URIs and cover **identity only** — Snap grants no access to
messages, contacts or shared content:

- `https://auth.snapchat.com/oauth2/api/user.display_name` — not user-toggleable
- `https://auth.snapchat.com/oauth2/api/user.external_id` — app-specific pseudonymous id, not
  user-toggleable
- `https://auth.snapchat.com/oauth2/api/user.bitmoji.avatar` — **user-toggleable**, so handle its
  absence gracefully
- `https://auth.snapchat.com/oauth2/api/camkit_lens_push_to_device` — automatic for Camera Kit apps

The Marketing API has a single coarse scope covering the entire ads surface. **Real authorization
comes from Business Manager roles, not scopes** — a token can carry the scope and still get a `403`
because the user's role is too narrow. Roles include `admin`, `member`, and since 2026-07-01
`agency_admin` and `agency_member`.

## Read the profile

`getUserProfile` → `GET https://kit.snapchat.com/v1/me` with the bearer token. Returns display
name, external id and Bitmoji, subject to the scopes granted.

Note the host: Login Kit profile reads go to `kit.snapchat.com`, not `adsapi.snapchat.com`.

## Keep the token alive

Access tokens expire after **3600 seconds**. Refresh with the `refresh_token` grant against the
same token endpoint. Refresh proactively on long-running jobs — a batch that outlives its token
fails midway, and because nothing on this API is idempotent, resuming is genuinely hard.

## Connect an agent to the Ads MCP server

Snap runs a hosted MCP server at **`https://mcp.snapchat.com/ads`**. It is read-only today; write
access is planned.

You do **not** register a client. Snap pre-registers one client id per agent vendor and rejects
anything else — dynamic client registration is not supported.

| Agent | Client id |
| --- | --- |
| Claude | `claude-snap-ads` |
| Codex | `codex-snap-ads` |
| ChatGPT | `chatgpt-snap-ads` |
| Antigravity | `antigravity-snap-ads` |
| Gemini CLI | `gemini-snap-ads` |

Claude Code, for example:

```
claude mcp add --transport http --client-id claude-snap-ads snap-ads https://mcp.snapchat.com/ads
claude mcp login snap-ads
```

Scope is `snapads.read` and it is load-bearing — omit it and the client requests a default set the
server rejects.

Access is two-stage: an **Admin or Business Admin** approves the agent for the organization once,
then every member authorizes individually. A member never gains a permission they did not already
have. Members revoke at <https://accounts.snapchat.com/v2/manage-apps>; the entry covers Ads MCP as
a whole, so removing it disconnects every agent that member connected.

## Debugging 401 and 403

| Symptom | Cause | Fix |
| --- | --- | --- |
| `401 Unauthorized` on a call that worked | Token past its 3600s TTL | Refresh, retry once |
| `401 Unauthorized - invalid client credentials` | Wrong `client_id`/`client_secret` | Re-check the developer portal |
| `400 invalid_grant` | Auth code reused, expired, or `redirect_uri` mismatch | Restart the authorization request |
| `403 Forbidden - insufficient permissions` | Business Manager role too narrow | Change the role, not the scope |
| `403 Forbidden - insufficient scopes` | Missing Login Kit scope | Re-authorize with the scope added |
| `401` from `mcp.snapchat.com/ads` with `WWW-Authenticate: Bearer resource_metadata=...` | Expected — you are unauthenticated | Complete the OAuth login for the agent |
