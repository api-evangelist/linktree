---
name: Report on Linktree profile and link performance
description: Pull account, link, and lifetime analytics for a Linktree profile and summarize performance.
api: mcp/linktree-mcp.yml
operations:
  - set_active_profile
  - get_account_analytics
  - get_analytics_lifetime_totals
  - get_top_links
  - get_profile_views
  - get_link_analytics
---

# Report on Linktree profile and link performance

## Before you start

- The only Linktree programmatic surface is the MCP server at `https://mcp.linktr.ee/mcp`
  (streamable-http). There is no public REST API.
- Authenticate with OAuth 2.0. Every tool needs its own `mcp:<tool_name>` scope; request only
  the scopes the flow below uses.
- If the user can access more than one profile and none is active, tools return
  `PROFILE_REQUIRED` with a `candidates[]` list. Call `set_active_profile` once with a
  `username` from that list, or pass `username`/`profileUrl` on every call.
- Write and destructive tools set the `x-linktree-require-approval` hint. Confirm with the
  user before executing them.
- On `RATE_LIMITED`, back off and retry. On `UPGRADE_REQUIRED`, the option needs a paid plan —
  tell the user rather than retrying.

## Steps

1. Resolve the target profile with `set_active_profile` (or pass `username`/`profileUrl`).
2. Call `get_analytics_lifetime_totals` for the all-time views, clicks, CTR, and earnings baseline.
3. Call `get_account_analytics` with the required `startDate` and `endDate` (`YYYY-MM-DD`) for the
   reporting window. Add `includeLinks`, `includeDevices`, `includeLocations`, `includeReferrers`,
   or `includeSubscribers` only if the user wants those breakdowns — each requires a paid plan.
4. Call `get_top_links` to rank every link by clicks, including links with zero clicks.
5. For any link the user wants to dig into, call `get_link_analytics`.
6. Use `get_profile_views` with `days` for a quick recent-window view count.
7. Summarize: totals, trend, best and worst performing links, and a concrete recommendation.

## Errors

- `UPGRADE_REQUIRED` — the requested breakdown or date range needs a paid plan. Report which
  option triggered it and offer the free-plan equivalent instead of retrying.
- `RATE_LIMITED` — back off before requesting further breakdowns.
