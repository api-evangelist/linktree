---
name: Update a Linktree profile's identity and appearance
description: Update display name, bio, appearance settings, and social icon links on a Linktree profile.
api: mcp/linktree-mcp.yml
operations:
  - set_active_profile
  - get_account
  - get_suggested_bio
  - update_account_profile
  - get_appearance
  - update_appearance
  - get_social_links
  - upsert_social_link
  - reorder_social_links
  - delete_social_link
---

# Update a Linktree profile's identity and appearance

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
2. Call `get_account` to read the current display name, bio, avatar, and username.
3. If the user wants help writing a bio, call `get_suggested_bio` and present the suggestion
   for approval rather than applying it directly.
4. Call `update_account_profile` to set the display name and/or bio. Use `clearBio` /
   `clearDisplayName` to clear a field rather than sending an empty string.
5. Call `get_appearance` to read current colors, fonts, button styles, and background,
   then `update_appearance` to change them. Always show the user the before/after.
6. For social icons: `get_social_links` to read the current set and order,
   `upsert_social_link` to create or update one for a supported platform,
   `reorder_social_links` to move one, and `delete_social_link` to remove one.
7. `update_account_profile`, `update_appearance`, `upsert_social_link` and
   `reorder_social_links` are writes, and `delete_social_link` is destructive —
   confirm each with the user first.

## Errors

- `INVALID_INPUT` — an unsupported platform or malformed appearance value was sent.
- `NOT_FOUND` — the social link being reordered or deleted does not exist.
