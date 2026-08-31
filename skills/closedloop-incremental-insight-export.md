---
name: closedloop-incremental-insight-export
description: Run a repeatable, resumable export of ClosedLoop AI product insights into a warehouse or file using the fixed-watermark cursor, and checkpoint it correctly so a failed run recovers without gaps.
api: ClosedLoop AI REST API
base_url: https://api.closedloop.sh/v1
base_url_eu: https://eu.api.closedloop.sh/v1
operations:
  - listInsights
  - getInsight
generated: '2026-08-30'
method: generated
source: >-
  Authored by API Evangelist from openapi/closedloop-public-api-openapi.yaml (v1.8.0) and
  https://closedloop.sh/docs/api-reference/conventions. Every operationId, parameter, response
  field and error code below appears in the published contract or the published Conventions page.
---

# Incremental insight export

Pull ClosedLoop AI insights on a schedule without re-reading the whole set and without losing rows when a run dies mid-way.

## Before you start

- You need a `/v1` API key from **Settings → API Keys** in the ClosedLoop AI app. Send it as `X-API-Key` on every request (`apikey` is accepted as an alias).
- **Pick the base URL by the key's region.** A key issued in the EU only works against `https://eu.api.closedloop.sh/v1`; a US key only against `https://api.closedloop.sh/v1`. There is no cross-region routing — the wrong host returns `401`.
- Keys are team-scoped. Everything you read is your team's data and nothing else's.
- The whole API is read-only. Nothing in this skill can modify or delete anything.

## Steps

### 1. First run — open the window with `updated_since`

```
GET /insights?updated_since=2026-05-01T00:00:00Z&limit=200
X-API-Key: <key>
```

Do **not** pass `offset` on an incremental run. `updated_since` selects the cursor mode; `offset` selects the other, non-resumable mode.

The response is ordered by `updated_at`, then `id`, and carries:

```json
{
  "data": ["..."],
  "pagination": {
    "limit": 200,
    "has_more": true,
    "next_cursor": "opaque-cursor",
    "sync_until": "2026-05-14T10:30:00.123456Z"
  }
}
```

`sync_until` is a **fixed upper watermark** captured at the start of the run. The export window is `updated_since < updated_at <= sync_until`, and it does not move while you page.

### 2. Page with `cursor` only

```
GET /insights?cursor=opaque-cursor&limit=200
X-API-Key: <key>
```

Send only `cursor` and, optionally, a different `limit`. Do not re-send the filters — the cursor already carries them. Treat the cursor as opaque: it is signed and bound to the team, the API key, the filters and the watermark.

There is no `total` on a cursor export, by design. `has_more` is what tells you when you are done.

### 3. Upsert by `id`, then checkpoint

Upsert every row by its immutable `id`. Insight ids are UUIDs that are never reassigned or recycled, so an upsert key is safe forever.

**Save `sync_until` as your next checkpoint only after the final page returns `has_more: false`.** Checkpointing early is the one way to lose rows: replaying the last committed window gives at-least-once recovery, but only if the committed window actually completed.

### 4. Next run

Pass the saved `sync_until` as the new `updated_since`. Repeat.

## Two things this export will not tell you

Both are stated by the provider and neither is a bug you can work around from the client side:

1. **Association-only changes may not appear.** A change made only to an insight's product, product-feature or product-area association may not advance its `updated_at`, so it may not surface until the insight itself changes. If association accuracy matters, periodically re-read affected insights with `getInsight` rather than relying on the stream.
2. **Deletions emit no tombstones.** A row that disappears from ClosedLoop AI will simply stop being updated. Your warehouse cannot learn it was removed from this stream. If you need removal detection, you need a periodic full reconciliation pass.

## Errors to handle

| Status | Code | What to do |
| --- | --- | --- |
| 400 | `VALIDATION_ERROR` | The cursor or a filter is invalid. Restart from the last committed `sync_until`. |
| 400 | `UNSUPPORTED_QUERY_PARAMETER` | You sent a parameter `/insights` does not accept. The message names it; the hint lists what is accepted. `/v1` rejects rather than ignores unknown parameters. |
| 401 | `NO_API_KEY` / `INVALID_API_KEY` | Missing, revoked, or wrong-region key. |
| 503 | `EXPORT_UNAVAILABLE` | The watermark could not be captured safely. Retry shortly and **do not** advance your checkpoint. |
| 503 | `API_KEY_AUTH_UNAVAILABLE` | Honour `Retry-After` — wait 10 seconds, then exponential backoff. |

## Key rotation mid-run

Cursors are bound to the exact API key that created them. If the key is rotated while an export is running, the cursor dies with it. Restart from the last fully committed `sync_until` using the new key.

## Rate limits

There is no general fixed per-key quota on the public API. The only documented throttling is a protective safeguard on *semantic* search, which this export does not use. Still handle `429` and honour `Retry-After`.
