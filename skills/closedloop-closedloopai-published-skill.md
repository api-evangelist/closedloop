---
name: Closedloopai
description: Use when building product intelligence workflows, querying customer evidence via API or MCP, integrating customer conversations into product decisions, monitoring recurring customer demand, or connecting product feedback to roadmap work. Agents should reach for this skill when working with product insights, themes, customer context, competitive intelligence, or when building dashboards and automations around customer evidence.
metadata:
    mintlify-proj: closedloopai
    version: "1.0"
---

# ClosedLoop AI Skill

## Product summary

ClosedLoop AI connects customer conversations and product behavior to traceable product insights. It ingests evidence from calls (Gong, Fireflies, Google Meet), messages (Slack, Intercom, Ada), surveys (Typeform), and webhooks, then extracts and organizes product feedback into searchable Insights, strategic Context, and Themes ranked by business impact. Agents use it to find what customers say about the product, validate source evidence, monitor new feedback with Watches, and connect recurring demand to Roadmap decisions.

Access the product via the web app at https://app.closedloop.sh, the REST API at `https://api.closedloop.sh/v1` (EU: `https://eu.api.closedloop.sh/v1`), or the MCP at `https://mcp.closedloop.sh` (EU: `https://eu.mcp.closedloop.sh`). API keys are created in **Settings → API Keys** and sent as the `X-API-Key` header. MCP connects to Claude Code, Claude Desktop, Cursor, Cowork, and other compatible AI tools for interactive product intelligence queries.

## When to use

Reach for this skill when:

- **Querying product evidence**: Search Insights (product problems), Context (strategic intelligence like churn, competition, satisfaction), or Themes (recurring problems ranked by business impact)
- **Building automations**: Pull insights, themes, customers, or analytics into dashboards, data warehouses, or scheduled jobs via the REST API
- **Integrating with AI tools**: Connect Claude or other MCP-compatible agents to query product intelligence, find reference customers, or analyze competitive mentions
- **Monitoring topics**: Create Watches to send new evidence matching a topic to your Inbox, then act on matches
- **Validating decisions**: Trace product feedback back to original customer quotes and source transcripts before using evidence in product decisions
- **Prioritizing work**: Evaluate Themes (recurring customer problems) with RIC scoring (Reach, Impact, Confidence) and customer impact data to inform Roadmap decisions
- **Connecting integrations**: Set up data sources (Gong, Slack, HubSpot, Salesforce, Linear, Jira) to bring conversations and CRM context into the product

## Quick reference

### Core concepts

| Term | What it is | When to use |
|------|-----------|------------|
| **Insight** | Individual product problem, request, or bug reported by a customer | Search when you need specific product feedback with severity and source |
| **Context** | Strategic intelligence: churn reasons, competitor mentions, satisfaction, buying behavior | Search when you need to understand *why* a customer is making a decision |
| **Behavior** | Product usage patterns from connected analytics (Amplitude, PostHog) | Search when you need to see what customers are actually doing in the product |
| **Theme** | Recurring problem: cluster of related Insights ranked by RIC score and customer impact | Use for Roadmap prioritization; shows business impact across multiple customers |
| **Watch** | Saved filter that sends new matching evidence to your personal Inbox | Create when you want to monitor a topic over time |

### API authentication

```bash
# Create key in Settings → API Keys, then send on every request:
curl https://api.closedloop.sh/v1/insights \
  -H "X-API-Key: clai_live_xxxxxxxxxxxxxxxxxxxx"

# EU workspace: use https://eu.api.closedloop.sh/v1
```

**Key properties**: Team-scoped (limited to your team's data) and region-scoped (EU key only works against EU endpoint).

### MCP setup by client

| Client | Setup |
|--------|-------|
| **Claude Code** | `claude mcp add --transport http closedloop-ai https://mcp.closedloop.sh` (EU: `https://eu.mcp.closedloop.sh`) |
| **Claude Desktop / Cowork** | Customize → Connectors → + → Custom → Web. Name: `ClosedLoop AI`, URL: `https://mcp.closedloop.sh` |
| **Cursor** | Add to `.cursor/mcp.json`: `{"mcpServers": {"closedloop-ai": {"url": "https://mcp.closedloop.sh"}}}` |
| **LiteLLM** | Create service client in Integrations → Developer tools → LiteLLM, copy config to `config.yaml` |

### Common API endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /insights` | List product insights with filters (category, severity, customer, date range) |
| `GET /insights/{id}` | Get full insight detail: pain point, workaround, competitor gap, verbatim quote |
| `GET /themes` | List recurring problems ranked by RIC score; supports semantic search |
| `GET /themes/{id}` | Get theme detail: severity breakdown, affected customers, supporting insights, features |
| `GET /customers` | Search customers by name, industry, country; includes CRM context and deal values |
| `GET /customers/{id}` | Get customer profile: deals, contacts, intelligence summary, strategic context |
| `GET /integrations` | List connected data sources and their sync status |

### Common MCP tools

| Tool | Purpose |
|------|---------|
| `get_overview` | High-level summary: total insights, deal blockers, churn risks, top categories |
| `search_insights` | Hybrid search (keyword + semantic) across product feedback |
| `search_opportunities` | Find Themes ranked by business impact; returns RIC scores and customer counts |
| `search_customers` | Find customers by name, industry, engagement; includes revenue context |
| `search_signals` | Search strategic intelligence: satisfaction, churn, competitor mentions, buying behavior |
| `get_trends` | Time-bucketed counts for trend analysis; supports filtering by category or feature area |

### Pagination and export

```bash
# Offset pagination (default)
curl "https://api.closedloop.sh/v1/insights?limit=50&offset=0" \
  -H "X-API-Key: clai_live_xxxxxxxxxxxxxxxxxxxx"

# Incremental export (for repeatable syncs)
curl "https://api.closedloop.sh/v1/insights?updated_since=2026-05-01T00:00:00Z&limit=200" \
  -H "X-API-Key: clai_live_xxxxxxxxxxxxxxxxxxxx"
# Save sync_until from response; use cursor for next page
```

## Decision guidance

### When to use Insights vs Context vs Behavior

| Question | Use | Why |
|----------|-----|-----|
| What product problem did a customer report? | **Insights** | Captures actionable product feedback with severity and source |
| Why is a customer at risk of churn? | **Context** | Captures strategic business situation (workaround, retention risk, competitor evaluation) |
| Are customers actually using a feature? | **Behavior** | Shows product usage patterns; does not explain intent |
| Should we build this feature? | **Themes** + **Insights** | Themes show recurring problems; Insights provide source evidence |

### When to use REST API vs MCP

| Scenario | Use | Why |
|----------|-----|-----|
| Scheduled job, ETL, unattended agent | **REST API** | API-key auth, no OAuth approval needed at runtime |
| Interactive Claude assistant, real-time queries | **MCP** | OAuth flow, natural language queries, integrated into agent workflow |
| Databricks, data warehouse sync | **REST API** | Designed for server-to-server workloads |
| Building PRD, analyzing customer evidence | **MCP** | Claude can reason over results, ask follow-up questions |

### When to create a Watch

| Situation | Create Watch | Why |
|-----------|--------------|-----|
| Topic you want to monitor over time | **Yes** | Watch sends new matches to Inbox; you review and act |
| One-time investigation | **No** | Just search Insights/Context; don't create recurring workflow |
| Topic is too noisy | **No** | Narrow filters first, then create Watch from focused view |
| Need to track multiple related topics | **Yes** | Create separate Watches for each topic; manage in Settings → Watches |

## Workflow

### Find and validate product evidence

1. **Choose your lens**: Open Insights (product problems), Context (strategic intelligence), or Behavior (usage patterns)
2. **Narrow the scope**: Search by customer, product area, or topic; apply filters (date range, severity, category)
3. **Open a result**: Read full detail, not just the list summary; confirm customer, date, source
4. **Trace to source**: Open the transcript or external source; verify the extracted insight matches what the customer said
5. **Choose next action**: Use evidence in Roadmap review, or create a Watch to monitor the topic

### Create a Watch and monitor Inbox

1. **Build a focused view**: Open Insights or Context; apply at least one filter (topic, customer, severity)
2. **Save the Watch**: Click **+ New watch**, enter a name, choose priority, review preview count, click **Save watch**
3. **Note**: Preview shows existing matches; only *new* evidence after save goes to Inbox
4. **Review matches**: Open Inbox, filter to **From watches**, review Critical matches under **Needs attention**
5. **Act on each match**: Open the match, inspect source evidence, then Snooze, Resolve, or Dismiss
6. **Tune the Watch**: Open Settings → Watches; pause, resume, or delete if too noisy

### Query product intelligence via API

1. **Create an API key**: Settings → API Keys → Create API Key; copy immediately (shown only once)
2. **Choose your endpoint**: `/insights` for product feedback, `/themes` for recurring problems, `/customers` for accounts
3. **Build your query**: Add filters (category, severity, date range, customer_id); use `limit` and `offset` for pagination
4. **Send the request**: Include `X-API-Key` header; parse JSON response
5. **For repeatable syncs**: Use `updated_since` and cursor-based pagination; save `sync_until` checkpoint

### Connect MCP to Claude

1. **Add the MCP**: Run `claude mcp add --transport http closedloop-ai https://mcp.closedloop.sh` (or use Customize → Connectors for Claude Desktop)
2. **Authorize**: Click the authorization link; log in to ClosedLoop AI; select workspace; click **Connect**
3. **Ask naturally**: Type questions like *"What are the top customer pain points?"* or *"Show me deal blockers for enterprise customers"*
4. **Claude queries tools**: Claude calls `search_insights`, `search_opportunities`, `get_customer`, etc. and reasons over results

### Prioritize work with Roadmap

1. **Start with Themes**: Open Roadmap → Themes; filter to **Uncovered** (not connected to planned work)
2. **Narrow by impact**: Search by product area; sort by RIC, Customers, Deal blockers, or Severity
3. **Validate the problem**: Open a Theme; review description, affected customers, source insights; confirm one coherent problem
4. **Interpret RIC**: Use Reach, Impact, Confidence to compare Themes; add strategy, effort, timing, opportunity cost
5. **Check planned work**: Open Product map → Features/Initiatives; verify scope addresses the Theme's problem
6. **Decide**: Build, experiment, investigate, defer, or reject; manage work in connected product management source

## Common gotchas

- **API key shown only once**: Copy immediately when created. If lost, revoke and create a new one.
- **Never expose API keys in client-side code**: Use only in server-side workloads, scheduled jobs, ETL pipelines.
- **Region-scoped keys**: EU key only works against `https://eu.api.closedloop.sh/v1`; US key only works against `https://api.closedloop.sh/v1`. Mismatch returns `401 INVALID_API_KEY`.
- **Watch preview is not backfilled**: Preview count shows existing matches; only *new* evidence after save goes to Inbox. Do not expect past matches to appear.
- **Unsupported query parameters are rejected**: The API rejects unknown filters with `400 UNSUPPORTED_QUERY_PARAMETER` rather than ignoring them. Check the endpoint documentation.
- **Incremental export cursors are opaque and key-bound**: If you rotate an API key during an export, restart from the last committed `sync_until` checkpoint with the new key.
- **Theme merges return `410 Gone`**: When themes are merged, `GET /themes/{old_id}` returns `410` with `replacement_theme_id`. Use the replacement ID.
- **Insight association changes may not advance `updated_at`**: A change to a product or feature association alone may not trigger an insight update. Wait for the insight itself to change.
- **Watches are personal**: Each Watch belongs to the creator's Inbox; other team members do not see it.
- **Deleting a Watch deletes its Inbox matches**: Pause instead if you want to keep history.
- **Data Log does not retry**: If an import shows Failed, contact support; there is no retry button.
- **Semantic search has rate limits**: `/themes` and `/features` share one semantic-search budget; either endpoint can return `429 RATE_LIMIT_EXCEEDED`. Honor `Retry-After`.

## Verification checklist

Before submitting work with ClosedLoop AI:

- [ ] **API key**: Created in Settings → API Keys; stored securely; never exposed in code
- [ ] **Region match**: API key region matches base URL (EU key → EU endpoint)
- [ ] **Authentication header**: Every request includes `X-API-Key` header
- [ ] **Evidence traced**: Product insight or Theme is traced back to original customer quote and source transcript
- [ ] **Filters applied**: Searches use appropriate filters (date range, severity, customer, category) to narrow scope
- [ ] **Pagination handled**: Offset pagination uses `limit` and `offset`; incremental exports use cursor and `sync_until` checkpoint
- [ ] **Watch filters focused**: Watch is created from a filtered view with at least one active filter; not too broad
- [ ] **MCP authorized**: Claude or other AI tool shows ClosedLoop AI tools available; test with `get_overview` or `search_insights`
- [ ] **Data Log checked**: If expected evidence is missing, verify import status in Data Log
- [ ] **RIC interpreted correctly**: RIC score informs Roadmap decision but does not make it; add strategy, effort, timing

## Resources

- **Comprehensive page listing**: https://closedloop.sh/docs/llms.txt
- **API Reference**: https://closedloop.sh/docs/api-reference/introduction — Start here for REST API authentication, pagination, filtering, error shapes
- **MCP Overview**: https://closedloop.sh/docs/mcp-server/overview — Setup and tool discovery for Claude, Cursor, Cowork, LiteLLM
- **Find and validate product evidence**: https://closedloop.sh/docs/guides/find-product-evidence — Step-by-step guide to search, narrow, trace, and act on evidence

---

> For additional documentation and navigation, see: https://closedloop.sh/docs/llms.txt