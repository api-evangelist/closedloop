---
name: closedloop-competitive-and-churn-signals
description: Pull competitor pressure and churn-risk evidence out of ClosedLoop AI and tie each mention back to the named customer and the deal behind it.
api: ClosedLoop AI REST API
base_url: https://api.closedloop.sh/v1
operations:
  - listCompetitors
  - listCompetitorMentions
  - listContext
  - listCustomers
  - getCustomer
  - listInsights
  - getInsight
generated: '2026-08-30'
method: generated
source: >-
  Authored by API Evangelist from openapi/closedloop-public-api-openapi.yaml (v1.8.0) and
  https://closedloop.sh/docs/api-reference/introduction. NOTE ON OPERATION IDS -- five of the seven
  operations named above (listCompetitors, listCompetitorMentions, listContext, listCustomers,
  getCustomer) carry NO operationId in the published spec; the ids used here are the ones supplied
  by overlays/closedloop-public-api-overlay.yaml. The PATHS are what the contract guarantees:
  GET /competitors, GET /competitors/mentions, GET /context, GET /customers, GET /customers/{id}.
---

# Competitive and churn signals

Answer "who are we losing to, and which accounts are at risk" from recorded customer evidence rather than desk research.

## Before you start

`X-API-Key` header, region-matched base URL, all `GET`s. See `conventions/closedloop-conventions.yml`.

## Steps

### 1. See who actually comes up

```
GET /competitors
```

Returns the competitors your customers mention, each with a mention trend and direction. Start here rather than with a name you assume matters — the list is built from what customers said, not from a configured watchlist you might be wrong about.

### 2. Read the verbatim mentions

```
GET /competitors/mentions?date_from=2026-01-01&date_to=2026-08-30
```

Searchable by competitor, customer, text or date, with source details, newest evidence first. This is where the actual words are.

### 3. Widen to the full strategic picture

```
GET /context?type=churn_reason&customer_id=<id>
GET /context?type=competitor_mention
GET /context?type=satisfaction
```

`/context` is the strategic layer: churn reasons, competitor mentions, satisfaction, decision criteria, purchase hesitation, win reasons, onboarding friction. Filter by `customer_id`, `type` or date.

There is no `GET /context/{id}` in the REST contract. If you need one record's full detail plus its source URL back to the original conversation, that is only reachable through the MCP `get_signal` tool — see `mcp/closedloop-tool-crosswalk.yml`.

### 4. Attach revenue and the account family

```
GET /customers?q=<name>
GET /customers/{id}
```

Customers carry CRM context — plan, ARR, active/churned — and an **account-family hierarchy** via `parent_id`: a parent account with its child properties. Roll risk up to the parent before you report a number, or you will count one enterprise account several times.

`GET /customers/{id}` adds the account summary, deals, people, churn state and a chronological activity timeline.

### 5. Join back to the product problem

```
GET /insights?customer_id=<id>&is_churn_risk=true
GET /insights/{id}
```

Insight and context records share the **same resolved `customer_id`**. That join is the whole point: a churn reason on its own is a complaint, but a churn reason lined up with the specific insight, its severity, and the deal value behind the account is a prioritisation input.

`getInsight` gives you `competitor_gap` — the explicit statement of what a competitor does that you do not — alongside the verbatim quote.

## Reporting rules

- **Quote from the detail call.** List responses carry previews; `GET /insights/{id}` carries the verbatim `quote`. Never present a truncated preview as a customer quote.
- **Say what the number counts.** Product-scoped filters return only insights with a recorded association; unassigned insights are excluded. Report "attributed mentions", not "all mentions".
- **Check the window.** `date_from` and `date_to` are inclusive on both ends and take ISO 8601 dates. `source_date` is when the feedback happened; `updated_at` is the export cursor timestamp — do not use `updated_at` as a "when did the customer say this" date.
- **Handle 410 on any stored theme id** you carry alongside this analysis: follow `replacement_theme_id`, which is always final.

## Errors

Same envelope everywhere: `{"error", "code", "hint"}`. Branch on `code`, not on the message string. `/v1` rejects unsupported query parameters with `400 UNSUPPORTED_QUERY_PARAMETER` rather than ignoring them, so a mistyped filter fails loudly instead of quietly returning unfiltered results — treat that 400 as a correction, not an outage.
