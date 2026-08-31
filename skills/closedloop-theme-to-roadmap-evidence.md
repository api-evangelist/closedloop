---
name: closedloop-theme-to-roadmap-evidence
description: Take a ClosedLoop AI Theme from a ranked list down to the customer quotes and buildable features behind it, checking evidence coverage first so a thin scope is not mistaken for a real signal.
api: ClosedLoop AI REST API
base_url: https://api.closedloop.sh/v1
operations:
  - listProducts
  - listAreas
  - getCoverage
  - listThemes
  - getTheme
  - listFeatures
  - getFeature
  - listInsights
  - getInsight
generated: '2026-08-30'
method: generated
source: >-
  Authored by API Evangelist from openapi/closedloop-public-api-openapi.yaml (v1.8.0) and
  https://closedloop.sh/docs/api-reference/conventions. Every operationId and parameter named here
  exists in the published contract.
---

# Theme to roadmap evidence

Go from "what should we build" to the actual customer quotes, without over-reading a scope that has too little evidence to support a decision.

## Before you start

Send `X-API-Key` on every request, against the base URL matching your key's region. Everything below is a `GET`; nothing here changes any state.

## Steps

### 1. Discover the product scope ids

```
GET /products          -> listProducts
GET /areas             -> listAreas
```

You need real ids, not names. Filters accept `product_id`, `product_feature_id`, `product_area_id` and `feature_area_id`, and insight responses expose the same ids back (`products[].id`, `product_features[].id`, `product_area_id`, `feature_area_id`), so scopes are round-trippable.

**Do not mix up two id kinds.** A *product feature* is a buildable item from `/features`. A *feature area* is the subject area an insight is filed under, from `/areas`. `product_feature_id` and `feature_area_id` are never interchangeable, and the field names invite exactly that mistake.

### 2. Check coverage before you trust the numbers

```
GET /coverage?product_id=<id>          -> getCoverage
```

This is the step most callers skip and it is the one that stops a wrong decision. `/coverage` tells you whether the evidence in a scope is informative, too thin, or predates coverage, and returns the coverage dates and windowed insight totals. A scope with three insights will still return a ranked Theme — coverage is how you learn not to act on it.

`search_mode` is **not** accepted on `/coverage`. Sending it returns `400 UNSUPPORTED_QUERY_PARAMETER`.

### 3. Rank the Themes

```
GET /themes?status=active&sort=ric_score&product_id=<id>    -> listThemes
```

Default search on `q` is **lexical**: the whole `q` value must occur as one case-insensitive substring in the title or description. For meaning-based matching:

```
GET /themes?q=payment%20failures%20and%20chargebacks&search_mode=semantic&min_similarity=0.50
```

`q` accepts at most 500 characters. `min_similarity` defaults to `0.50` and accepts `0.30`–`1.00`. Semantic results are ordered by similarity then theme id, so **semantic relevance overrides `sort`** — do not expect your `sort` to survive.

If embedding generation fails you get `503 SEMANTIC_SEARCH_UNAVAILABLE`. Add `allow_fallback=true` to permit lexical fallback, then read the response's `search_mode` and `min_similarity`: a fallback returns `"search_mode": "lexical"` and `"min_similarity": null`, so you can tell you did not get what you asked for.

### 4. Open the Theme

```
GET /themes/{id}       -> getTheme
```

Returns the severity breakdown, affected customers, supporting insights, and the buildable `features` under the theme — everything needed for the decision in one call.

**Handle 410.** If the theme was merged, you get `410 THEME_RETIRED` with `replacement_theme_id`. That pointer is **final**, never a chain link, so re-request once against the replacement and you are done. To list current themes alongside merged ones, use `GET /themes?include_retired=true`.

### 5. Drill to the buildable feature

```
GET /features?theme_id=<id>            -> listFeatures
GET /features/{id}                     -> getFeature
```

**Watch the count semantics.** When `/features` is filtered by `feature_area_id` or `product_area_id`, `insight_count` and `unique_customer_count` are **scoped to that area**, and features with no evidence in the area are omitted. `GET /features/{id}` is never scoped — it always reports the all-areas figure. The same field name can legitimately return different numbers from the two calls. Also note that `sort=ric_score` stays the feature's stored overall score under an area filter, while `sort=insight_count` and `sort=unique_customer_count` rank on the scoped values.

### 6. Read the actual words

```
GET /insights?theme-scoped filters&limit=200    -> listInsights
GET /insights/{id}                              -> getInsight
```

`getInsight` is where the verbatim `quote`, `pain_point`, `workaround`, `competitor_gap`, `willingness_to_pay`, `use_case`, `business_outcomes` and `kano_category` live. Quote from `getInsight`, not from a list preview.

### 7. Add the customer's strategic picture

```
GET /context?customer_id=<id>
```

Insight and context records carry the **same resolved `customer_id`**, so you can line up what a customer said about the product with why they are churning, who they compared you to, and how satisfied they are.

## Coverage honesty

Product filtering returns only insights with a *recorded* association. Some insights stay unassigned when no reliable automatic match is found, so a scoped result may omit relevant evidence. A fully unassociated insight returns both `products` and `product_features` empty. Say so when you report a count — the number is "insights we could attribute", not "insights that exist".

## Rate limits

`/themes` and `/features` share **one** semantic-search budget and are not separately allowanced, so heavy semantic traffic on either can produce a `429 RATE_LIMIT_EXCEEDED` on the other. Honour `Retry-After`. Note that only `/themes` declares the 429 in the published spec; `/features` can return it anyway.
