---
name: snapchat-report-performance
description: >-
  Pull Snapchat advertising performance at any level of the hierarchy — account, campaign, ad squad
  or ad — and walk the full inventory with cursor pagination without dropping or double-counting
  entities. Use when asked how a Snapchat campaign is performing, to build a report, or to audit
  spend across an account.
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conventions/snapchat-conventions.yml
api: Snapchat Marketing API
base: https://adsapi.snapchat.com/v1
operations:
  - listOrganizations
  - listAdAccounts
  - listCampaigns
  - listAdSquadsByCampaign
  - listAdsByCampaign
  - getAdAccountStats
  - getCampaignStats
  - getAdSquadStats
  - getAdStats
---

# Report on Snapchat ad performance

Stats are not a standalone resource on this API. There is no `/stats` collection — you ask each
level of the hierarchy for its own numbers.

## Pick the level first

| Question | Operation | Path |
| --- | --- | --- |
| How is the whole account doing? | `getAdAccountStats` | `GET /adaccounts/{ad_account_id}/stats` |
| How is this campaign doing? | `getCampaignStats` | `GET /campaigns/{campaign_id}/stats` |
| Which targeting is working? | `getAdSquadStats` | `GET /adsquads/{ad_squad_id}/stats` |
| Which creative is working? | `getAdStats` | `GET /ads/{ad_id}/stats` |

Start at the coarsest level that answers the question. Every level down multiplies your call count
against a 10 req/s per-token ceiling.

## Walk the hierarchy

1. `listOrganizations` → `GET /me/organizations`
2. `listAdAccounts` → `GET /organizations/{organization_id}/adaccounts`
3. `listCampaigns` → `GET /adaccounts/{ad_account_id}/campaigns`
4. `listAdSquadsByCampaign` → `GET /campaigns/{campaign_id}/adsquads`
5. `listAdsByCampaign` → `GET /campaigns/{campaign_id}/ads`

`listAdsByCampaign` flattens across a campaign's ad squads, so prefer it over looping
`listAdsByAdSquad` when you want the whole campaign. `listAdsByAdAccount` and
`listAdSquadsByAdAccount` do the same at account level.

## Paginate correctly

Every list above is cursor paginated:

- Send `?limit=<50..1000>`. Values outside that range are rejected.
- The response carries `paging.next_link` — a **fully formed absolute URL** with an opaque
  `cursor`. Follow it verbatim. Do not rebuild it, and do not try to decode the cursor.
- Stop when `paging.next_link` is absent.
- Paginated results are ordered by `CreatedAt`. **Non-paginated calls are unsorted** — if you skip
  `limit` you also lose ordering guarantees.

Use `limit=1000` for bulk reporting. Fewer round trips is the only rate-limit lever you have.

## Respect the ceiling

Two caps apply simultaneously: **20 requests/second across the whole application** and
**10 requests/second per access token**. Spreading work across more tokens does not get you past
the app-wide cap.

There is no `X-RateLimit-*` header and no `Retry-After`. You cannot see how close you are — you
only find out by receiving a `429`. Rate-limit yourself client-side, and on a `429` back off
exponentially with jitter.

## Async reports

Large report requests are asynchronous. Generated reports stay downloadable for **7 days** after
generation, so re-request rather than assuming an old link still resolves.

## Read the envelope, not the status line

Collection responses carry `request_status` for the request and `sub_request_status` per entity. A
`200` can contain individually failed entities. Filter on `sub_request_status == "SUCCESS"` before
you aggregate, or your totals will silently under-report.

Keep `request_id` from anything that fails.

## Dimensions worth knowing

Insights `report_dimension` supports DMA and REGION, and as of 2026-07-09 those are no longer
US-only. Check `changelog/snapchat-changelog.yml` before assuming a dimension is unavailable — this
API's capabilities move without a version bump, because versioning is frozen at `v1`.
