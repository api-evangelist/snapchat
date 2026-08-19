---
name: snapchat-send-conversion-events
description: >-
  Send server-to-server web, app or offline conversion events to Snap's Conversions API (CAPI v3),
  validate them against the test endpoint first, hash PII correctly, and deduplicate against the
  Snap Pixel. Use when asked to integrate Snapchat conversion tracking, send purchase or signup
  events to Snap, or debug why CAPI events are being rejected.
generated: '2026-08-13'
method: generated
source: >-
  openapi/snapchat-conversion-events-api-openapi.yml,
  https://developers.snap.com/marketing-api/Conversions-API/Introduction,
  sandbox/snapchat-sandbox.yml
api: Snapchat Conversions API
base: https://tr.snapchat.com/v3
operations:
  - sendWebConversionEvents
  - sendAppConversionEvents
---

# Send conversion events to Snap

The Conversions API is a different animal from the Marketing API. Different host
(`tr.snapchat.com`, not `adsapi.snapchat.com`), different version (`v3`, not `v1`), different error
shape, and it is the one place Snap gives you a real test path.

## Choose the operation

| Event source | Operation | Path |
| --- | --- | --- |
| Website | `sendWebConversionEvents` | `POST /{pixel_id}/events` |
| Mobile app | `sendAppConversionEvents` | `POST /{snap_app_id}/events` |

`pixel_id` and `snap_app_id` are provisioned in Ads Manager. Neither has a CRUD endpoint — you
cannot create one through the API.

## Authenticate

Two options:

- **OAuth 2.0 bearer token** — `Authorization: Bearer <access_token>`. Preferred.
- **Long-lived CAPI token** from the Business Details page in Ads Manager, passed as
  `?access_token=<token>`.

The long-lived token is convenient and it is a query parameter, which means it lands in access
logs, proxy logs and browser history. Use the OAuth flow wherever the integration can carry it.
Either way the token must belong to the organization that owns the asset id in the path.

## Validate before you ship

Snap has no sandbox, but it does have a validate path:

```
POST https://tr.snapchat.com/v3/{asset_id}/events/validate
```

Same payload, same credentials, near-real-time feedback, and **no production conversion is
recorded**. Run every new payload shape through it first. Results are also visible in Events
Manager.

Two companion calls exist in the Java Business SDK for reading back what you sent:
`getValidateEventLogs(assetId)` (a summary of the last day) and `getValidateEventStats(assetId)`.

## Build the payload

Required on every event:

- `event_name` / `event_type`
- `event_time` / `timestamp`
- `action_source` / `event_conversion_type`
- `event_source_url` / `page_url`

**At least one identifier is mandatory.** Snap recommends sending all of them:

- `em` — hashed email
- `ph` — hashed phone number
- `client_ip_address` **and** `client_user_agent` together
- `madid` — mobile advertising id, app events only

If a first-party `_scid` cookie is available, pass it as `sc_cookie1`. Snap says this materially
increases match rate.

## Hash the PII

`em`, `ph` and the other hashing-required user-data parameters are accepted **only as hashed
values**. Sending a raw email address returns `INVALID` / `400`. Normalize before hashing —
lowercase, trim — per Snap's Data Hygiene guidance, or the hash will not match.

Never log the pre-hash values.

## Deduplicate

If the Snap Pixel fires in the browser *and* your server sends the same conversion through CAPI,
Snap counts it twice unless you supply deduplication signals. The same applies to an MMP postback
alongside direct CAPI for an app.

This is **event deduplication, not request idempotency**. It stops double-counting in reporting; it
does not make a retried HTTP POST safe. Those are different problems.

## Batch and send

Payload is `{"data": [ ...events... ]}`. Batch rather than sending one event per request — the
Marketing API rate ceiling logic applies here too and a `429` is a documented response on both
operations.

The Java Business SDK exposes `sendEvent`, `sendEventsBatch`, `sendEventAsync` and
`sendEventsBatchAsync` (`com.snap.business.sdk.v3:snap-capi`, current release **1.2.1** — note
Snap's own docs still tell you to depend on 1.1.5).

## Read the response

The Conversions API does **not** use the Marketing API envelope. It returns:

```json
{ "status": "INVALID", "test_event": true, "reason": "...", "event_logs": [ { "event": 1, "status": "INVALID", "errors": { "codes": ["517"] } } ] }
```

Check `status`, then walk `event_logs` — a batch can be partially invalid.

## Rejection reasons, all returning `400 / INVALID`

- Any required field missing
- Missing `price` and/or `currency` on a Purchase event
- Invalid hashed IP address
- Timestamp invalid or out of range
- Unhashed PII
- Insufficient PII

## Migrating from v2

Snap publishes a v2 → v3 migration guide. v2 is the superseded generation; new integrations should
target v3 directly. The Python Business SDK (`snap-business-sdk` on PyPI, last released 2022) still
targets v2 — use the Java SDK or call v3 directly.
