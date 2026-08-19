---
name: snapchat-launch-campaign
description: >-
  Stand up a complete Snapchat ad from nothing — find the organization and ad account, confirm a
  funding source exists, upload the media, build the creative, then create the campaign, ad squad
  and ad in the order the API requires. Use when asked to launch, create or set up a Snapchat
  advertising campaign programmatically.
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conventions/snapchat-conventions.yml, errors/snapchat-problem-types.yml
api: Snapchat Marketing API
base: https://adsapi.snapchat.com/v1
operations:
  - listOrganizations
  - listAdAccounts
  - listFundingSources
  - createMedia
  - uploadMedia
  - createCreative
  - createCampaign
  - createAdSquad
  - createAd
---

# Launch a Snapchat ad campaign

The Snapchat Marketing API enforces a strict containment order. You cannot create an ad squad
before its campaign, and you cannot attach a creative that has no uploaded media behind it. Work
top-down.

## Before you start

- Authenticate with OAuth 2.0. Send `Authorization: Bearer <access_token>` on every call. Tokens
  expire after **3600 seconds** — refresh against
  `https://accounts.snapchat.com/login/oauth2/access_token` rather than letting a long run die
  mid-sequence.
- **There is no idempotency key on this API.** If a `POST` times out you do not know whether it
  landed. Before retrying any create, re-run the matching list operation and check whether the
  entity already exists. Blind retries produce duplicate campaigns that spend real money.
- **There is no sandbox.** Every call in this skill writes to production. Create the campaign in a
  `PAUSED` status and only activate it after a human has reviewed it.

## Step 1 — Locate the organization

`listOrganizations` → `GET /me/organizations`

Returns the organizations the authorizing user belongs to. Take `organization_id`.

A `403 Forbidden - insufficient permissions` here means the user's Business Manager role is too
narrow — the fix is a role change in Business Manager, not a different scope.

## Step 2 — Locate the ad account

`listAdAccounts` → `GET /organizations/{organization_id}/adaccounts`

Take `ad_account_id`. This is the id almost every later call hangs off.

Use `getAdAccount` → `GET /adaccounts/{ad_account_id}` if you already have the id and only need to
confirm it is reachable.

## Step 3 — Confirm the account can pay

`listFundingSources` → `GET /organizations/{organization_id}/fundingsources`

An ad account with no active funding source will accept a campaign and then fail to deliver.
Check this before you build anything.

## Step 4 — Upload the media

Two calls, in order:

1. `createMedia` → `POST /adaccounts/{ad_account_id}/media` with `application/json`. Returns a
   `media_id`.
2. `uploadMedia` → `POST /media/{media_id}/upload` with `multipart/form-data` — the actual bytes.

These two are not atomic and neither is idempotent. If step 2 fails, reuse the same `media_id`
rather than calling `createMedia` again.

`400 Bad request - invalid file or exceeds size limit` means the asset violates Snap's media
specifications. Long-form video has its own chunking rules.

## Step 5 — Build the creative

`createCreative` → `POST /adaccounts/{ad_account_id}/creatives`, referencing the `media_id`.

Returns `creative_id`. Note there is **no delete operation for creatives** — a mistake here is
permanent clutter in the account, so validate before you post.

## Step 6 — Create the campaign

`createCampaign` → `POST /adaccounts/{ad_account_id}/campaigns`

Carries the objective and the budget. Returns `campaign_id`.

Set the campaign to `PAUSED` on creation.

## Step 7 — Create the ad squad

`createAdSquad` → `POST /adaccounts/{ad_account_id}/adsquads`, referencing `campaign_id`.

This is where targeting, placement, schedule and bid strategy live. Returns `ad_squad_id`.

If you need an audience segment, create it first with `createAudienceSegment` →
`POST /adaccounts/{ad_account_id}/segments` and reference it from the targeting block.

## Step 8 — Create the ad

`createAd` → `POST /adsquads/{ad_squad_id}/ads`, referencing `creative_id`.

Returns `ad_id` with a `review_status`. A newly created ad is not live until Snap's review passes.

## Step 9 — Verify

Read back the whole chain and confirm each entity exists exactly once:

- `getCampaign` → `GET /campaigns/{campaign_id}`
- `getAdSquad` → `GET /adsquads/{ad_squad_id}`
- `getAd` → `GET /ads/{ad_id}` and check `review_status`

## Reading the responses

Snap wraps everything in an envelope. **A `200` does not mean success.** Check
`request_status` for the whole request and `sub_request_status` on each entity inside it. Keep
`request_id` from any failing response — it is what Snap support asks for.

## When things fail

| Status | Meaning | Do this |
| --- | --- | --- |
| 400 | Bad request | Validate enums and required attributes against the entity reference. PATCH responses carry sub-cause strings and `E2023` for immutable fields. |
| 401 | Unauthorized | Token expired (3600s TTL). Refresh, then retry once. |
| 403 | Insufficient permissions | Business Manager role problem, not a scope problem. |
| 404 | Entity not found | Check you are using the right id — every id is a bare UUID with no type prefix, so it is easy to pass an ad squad id where an ad id belongs. |
| 429 | Rate limited | 20 req/s per app, 10 req/s per token, both enforced at once. There is **no `Retry-After` header** — back off exponentially with jitter. |

Full catalogue: `errors/snapchat-problem-types.yml`. Cross-cutting semantics:
`conventions/snapchat-conventions.yml`.
