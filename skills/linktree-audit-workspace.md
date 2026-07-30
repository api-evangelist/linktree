---
name: Audit a Linktree workspace and its profiles
description: Enumerate workspaces, their member profiles and members, and compare profile performance.
api: mcp/linktree-mcp.yml
operations:
  - get_workspaces_for_user
  - get_workspace
  - get_workspace_profiles
  - get_workspace_members
  - get_workspace_profiles_analytics
---

# Audit a Linktree workspace and its profiles

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

1. Call `get_workspaces_for_user` to list every workspace the authenticated user belongs to.
2. For the workspace of interest, call `get_workspace` for its details.
3. Call `get_workspace_profiles` to enumerate every Linktree profile in that workspace.
4. Call `get_workspace_members` to list who has access.
5. Call `get_workspace_profiles_analytics` for lifetime analytics across every profile in the
   workspace — use this rather than looping per-profile analytics calls.
6. Summarize: profile count, member count, top and bottom performing profiles, and any profile
   with no links or no traffic.

## Notes

- All five tools are read-only; no approval prompt is needed.
- Pass `workspaceUuid` to narrow other profile-targeting tools to this workspace.
