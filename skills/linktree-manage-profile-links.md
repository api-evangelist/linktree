---
name: Manage the links on a Linktree profile
description: Add, reorder, update, and remove links on a Linktree profile via the Linktree MCP server.
api: mcp/linktree-mcp.yml
operations:
  - list_accessible_profiles
  - set_active_profile
  - get_links
  - add_link
  - update_link
  - reorder_link
  - delete_link
---

# Manage the links on a Linktree profile

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

1. Call `list_accessible_profiles` to see which profiles the user can edit
   (pass `editableOnly: true` to filter). If exactly one matches, you may pass `setActive: true`.
2. Otherwise call `set_active_profile` with the chosen `username`.
3. Call `get_links` to read the current ordered link list, including link-app metadata.
   Always read before writing so you can reference real link identifiers.
4. To add a link, confirm the title and URL with the user, then call `add_link`.
5. To change an existing link's title, URL, or active state, call `update_link`.
6. To change display order, call `reorder_link`.
7. To remove a link, confirm explicitly with the user, then call `delete_link`
   (this is a destructive tool and cannot be undone through the MCP surface).
8. Re-read with `get_links` to confirm the resulting order and state.

## Errors

- `NOT_FOUND` — the referenced link no longer exists; re-read with `get_links`.
- `INVALID_INPUT` — a required parameter was missing or malformed.
- `PROFILE_REQUIRED` — no target profile resolved; go back to step 2.
