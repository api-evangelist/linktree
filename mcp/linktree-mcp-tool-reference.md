# Linktree MCP — Tool Reference

Manage Linktree profiles, links, appearance, and analytics via the Model Context Protocol.

- **Endpoint:** `https://mcp.linktr.ee/mcp` (transport: `streamable-http`, protocol `2025-03-26`)
- **Auth:** OAuth 2.0 (Descope); discover the authorization server via [protected-resource metadata](https://mcp.linktr.ee/.well-known/oauth-protected-resource). One grant covers every tool — request no `scope` (or only `offline_access` for a refresh token); per-tool scopes are not used.
- **Machine-readable manifest:** [https://mcp.linktr.ee/.well-known/mcp/tools.json](https://mcp.linktr.ee/.well-known/mcp/tools.json) · **llms.txt:** [https://mcp.linktr.ee/llms.txt](https://mcp.linktr.ee/llms.txt)

This server exposes **36 tools**. Tools tagged _approval required_ perform writes/deletes and set the `x-linktree-require-approval` hint so clients can prompt the user first.

## Profile targeting

Most tools accept optional `username`, `profileUrl`, `query`, and `workspaceUuid` parameters to target a specific accessible profile. If the caller has multiple accessible profiles and none is active, the tool returns a `PROFILE_REQUIRED` error (see [Error contract](#error-contract)); call `set_active_profile` once, or pass a target on each call.

## Tools

### `get_profile_sharing_guidance`

Get the selected Linktree profile URL and guidance for putting it in a social bio or website field. Use when the creator asks how to share their Linktree or, after a successful link update, when bio placement is relevant to their stated traffic goal. Sharing means adding the Linktree profile URL to the social profile, not drafting captions or posts. This tool does not edit social profiles or verify bio placement. Do not claim the URL has been added. If the creator already has the correct URL in their bio, no change is needed. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `platform` | instagram | tiktok | youtube | facebook | x | linkedin | other | no | The social platform where the creator wants to put their Linktree profile URL in their bio or website field. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `list_accessible_profiles`

Retrieve every Linktree profile the authenticated user can access across direct accounts and workspaces. Pass setActive: true only when the filters resolve to exactly one profile; otherwise call set_active_profile with a username from the returned candidates.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `allPages` | boolean | no | If true, return every matching profile and ignore page/limit. |
| `editableOnly` | boolean | no | If true, only return profiles the authenticated user can edit. |
| `limit` | number | no | Page size for paginated results. Defaults to 100. |
| `page` | number | no | 1-based page number for paginated results. Defaults to 1. |
| `query` | string | no | Optional search query. Matches username or display name. Username or profileUrl is still preferred for follow-up writes. |
| `setActive` | boolean | no | If true, persist the single resolved profile as the active profile for this conversation. Ignored unless exactly one profile matches. |
| `workspaceUuid` | string | no | Optional workspace UUID to limit discovery to one workspace. |

### `set_active_profile`

Set the active Linktree profile for the current conversation. Use this once after a PROFILE_REQUIRED response, or pass username/profileUrl directly on individual profile-specific calls.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_account`

Retrieve a Linktree profile's details (display name, bio, avatar, username). If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `update_account_profile`

Update a Linktree profile's display name and/or bio. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `bio` | string | no | Optional bio/description to set on the account. Use null to clear it. |
| `clearBio` | boolean | no | Explicitly clear the bio/description on the account. |
| `clearDisplayName` | boolean | no | Explicitly clear the display name/page title on the account. |
| `displayName` | string | no | Optional display name/page title to set on the account. Use null to clear it. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_suggested_bio`

Retrieve a suggested bio for a Linktree profile based on its existing content. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `seed` | number | no | Deterministic seed for reproducible suggestions. |
| `temperature` | number | no | Sampling temperature (0-2, default 0.7). |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_account_analytics`

Retrieve account-level analytics for a Linktree profile, including views, clicks, CTR, and optional breakdowns. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `endDate` | string | yes | End date in YYYY-MM-DD format. |
| `startDate` | string | yes | Start date in YYYY-MM-DD format. |
| `includeDevices` | boolean | no | Include device breakdown data. |
| `includeLinks` | boolean | no | Include individual link performance data. |
| `includeLocations` | boolean | no | Include location breakdown data. |
| `includeReferrers` | boolean | no | Include referrer breakdown data. |
| `includeSubscribers` | boolean | no | Include audience/subscriber data when available. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- View Custom Date Ranges: Free accounts are limited to the SDK free-plan analytics date range.
- Device-Based Analytics: includeDevices requests device-based analytics.
- Location-Based Analytics: includeLocations requests location-based analytics.
- Individual Link Analytics: includeLinks requests individual link analytics.
- Audience - Linktree Subscribe: includeSubscribers requests subscriber data.

### `get_analytics_lifetime_totals`

Retrieve lifetime analytics totals for a Linktree profile, including views, clicks, CTR, and earnings. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_top_links`

Retrieve click counts for all links on a Linktree profile, including links with zero clicks, ranked by performance. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_profile_views`

Retrieve profile views for a Linktree profile over a recent time window. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `days` | number | no | Number of days to look back. Defaults to 7. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- View Custom Date Ranges: The days parameter can request analytics outside the SDK free-plan date range.

### `get_social_integrations`

Retrieve the active social integrations connected to a Linktree profile. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_appearance`

Retrieve a Linktree profile's appearance settings, including colors, fonts, button styles, and background. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `update_appearance`

Update a Linktree profile's appearance, including colors, fonts, button styles, and background styling. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `backgroundColor` | string | no | Background color of the page as a 6-digit hex color code. |
| `backgroundGradientStyle` | string | no | Preset gradient to use when the background style is GRADIENTSTILL or GRADIENTANIMATED: GREEN, NOIR, ORANGE, PINK, RAINBOW, BLOOM, TWILIGHT, RISE, or GLOW. CUSTOM builds the gradient from the background color instead. |
| `backgroundStyle` | string | no | Background style for the profile: FLAT (solid color), BLUR, COLORUP, COLORDOWN, or GRADIENTRADIAL (gradients built from the background color), GRADIENTSTILL or GRADIENTANIMATED (preset gradients, chosen with backgroundGradientStyle), or the GRID, MORPH, ORGANIC, or MATRIX patterns. |
| `buttonCornerStyleType` | SHARP | CURVED | ROUND | ROUNDED_NONE | ROUNDED_SM | ROUNDED_LG | ROUNDED_FULL | no | Button corner style: ROUNDED_NONE (square), ROUNDED_SM, ROUNDED_LG, or ROUNDED_FULL (pill). The older SHARP, CURVED, and ROUND are still accepted and match ROUNDED_NONE, ROUNDED_LG, and ROUNDED_FULL. |
| `buttonShadowStyleType` | SOFT | HARD | NONE | SHADOW_NONE | SHADOW_SM | SHADOW_LG | SHADOW_FULL | no | Button shadow style: SHADOW_NONE, SHADOW_SM, SHADOW_LG, or SHADOW_FULL. The older NONE, SOFT, and HARD are still accepted and match SHADOW_NONE, SHADOW_SM, and SHADOW_FULL. |
| `buttonStyleType` | FILL | OUTLINE | GLASS | no | Button style type (FILL, OUTLINE, or GLASS). |
| `fontStyle` | string | no | Body font for the page text. Must be one of the supported fonts (use the exact name): Albert Sans, Belanosima, Bricolage Grotesque, DM Sans, Epilogue, IBM Plex Sans, Inter, Lato, Link Sans, M Plus Rounded, Manrope, Oxanium, Poppins, Red Hat Display, Roboto, Rubik, Space Grotesk, Syne, BioRhyme, Bitter, Caudex, Corben, Domine, Hahmlet, IBM Plex Serif, Lora, Merriweather, Noto Serif, Old Standard TT, PT Serif, Playfair Display, Roboto Serif, Roboto Slab, Source Serif Pro, IBM Plex Mono, Space Mono, Shantell Sans. Do not invent fonts — values outside this list are ignored. |
| `headingFont` | string | no | Heading font for the page text. Must be one of the supported fonts (use the exact name): Alfa Slab One, Anton, Belanosima, BioRhyme, Black Sansa, Chango, Chillax, Domine, Fustat, Gasoek One, IBM Plex Sans, Kavivanar, Link Sans, Manrope, Misto, Monofett, Old Standard TT, Oxanium, Playfair Display, Poppins, Roboto, Roboto Slab, Salsa, Sonder, Space Mono, Summer Glow, Viaoda Libre, Wagon, Albert Sans, Bricolage Grotesque, DM Sans, Epilogue, Inter, Lato, M Plus Rounded, Red Hat Display, Rubik, Space Grotesk, Syne, Bitter, Caudex, Corben, Hahmlet, IBM Plex Serif, Lora, Merriweather, Noto Serif, PT Serif, Roboto Serif, Source Serif Pro, IBM Plex Mono, Shantell Sans. Use "default" to match the body font. Do not invent fonts — values outside this list are ignored. |
| `headingSize` | normal | large | no | Heading size. Linktree sets it from the profile layout: hero → large, cutout/shape → extra large (not settable here), other layouts → normal. A size that doesn't match the layout is reported in unappliedFields. |
| `heroMode` | boolean | no | Enable hero avatar mode for the profile. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- Header Layout - Hero: heroMode requests the hero header layout feature.
- Gradients: Gradient background options require gradient background access.
- Backgrounds: Pattern background options require background feature access.
- Custom Title Font: headingFont requests title font customization.

### `get_knowledge_base`

Retrieve a Linktree profile's knowledge base entries (business, product, policy, or FAQ information). If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `searchQuery` | string | no | Optional search query to filter saved information by keyword. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- AI Chatbot: Creator knowledge entries are configured through the AI Chatbot feature.

### `pull_recommendations`

Use this tool whenever the creator asks for suggestions or ideas, or asks what to do, improve or change on their Linktree. If they mention a focus, pass it as `topic`. Retrieve up to three available recommendations for a Linktree profile. Treat each recommendation as input for reasoning, not as text to paste to the creator: understand what it changes, use public read tools such as get_links, get_account, get_appearance, get_social_links, and get_link_analytics for context when needed, explain it in your own words, and propose a concrete action. Only use write tools such as add_link, update_link, upsert_social_link, and update_appearance after the creator agrees. `items` is the same recommendations as unified items (id, status, headline, summary, why, facts, action); prefer it when present. Track the creator's decision on each item and always submit it afterward with submit_recommendation_feedback (dismiss_all if they are not interested in a recommendation). If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | number | no | Maximum number of recommendations to retrieve. Defaults to 3. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `topic` | string | no | What the creator wants help with, in their words (e.g. 'grow my newsletter', 'sell more merch'). Omit for general suggestions. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `submit_recommendation_feedback`

Record the creator's decision on a recommendation returned by pull_recommendations. Always call it after the creator acts on or declines a pulled recommendation, including when they decline. Send one item per recommendation item you discussed, using the item ids and kinds from the recommendation, with outcome acted (adopted as suggested), acted_with_edits (adopted with changes) or dismissed. For acted items, set link_id to the id of the link you actually created or updated. Example: a recommendation suggests 5 products and the creator adds 2 and skips 3, so send 2 items with outcome acted, signal added_to_profile and the new link_id, plus 3 items with outcome dismissed and signal explicit_dismiss. If the creator is not interested in the recommendation at all, set dismiss_all to true (items may then be empty). Only tell the creator feedback was recorded when this tool confirms it. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `recommendation_id` | string | yes | Id of the recommendation returned by pull_recommendations. |
| `recommendation_type` | string | yes | Type of the recommendation returned by pull_recommendations. |
| `dismiss_all` | boolean | no | Set to true when the creator is not interested in this recommendation. |
| `items` | object[] | no | One entry for each recommendation item the creator acted on or declined. |
| `note` | string | no | Optional note about the creator’s decision. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_workspaces_for_user`

Retrieve every workspace the authenticated user is a member of. Returns role, id, uuid, name, and avatarUrl for each workspace. Use the uuid from this response when calling other workspace tools.

- **Access:** read · **Approval:** none · **Identity:** user

No parameters.

### `get_workspace`

Retrieve details for a workspace the authenticated user belongs to, including the current user's role (OWNER, MANAGER, or MEMBER), id, uuid, name, and avatarUrl. Optionally pass a workspaceId to target a specific workspace; omit it to fetch the default workspace.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Optional workspace ID. If omitted, returns the default workspace for the current user. |
| `workspaceUuid` | string | no | Optional workspace UUID alias for workspaceId. |

### `get_workspace_profiles`

Retrieve every Linktree profile belonging to a workspace. Returns ids, names, avatars, access types, owner/admin summaries, and pending transfer state for each profile. Fetches all pages automatically. Call get_workspaces_for_user first to obtain the workspaceUuid.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Alias for workspaceUuid. Pass either workspaceId or workspaceUuid. |
| `workspaceUuid` | string | no | The UUID of the workspace to fetch profiles for. |

### `get_workspace_profiles_analytics`

Retrieve lifetime analytics for every Linktree profile in a workspace in a single batch call. Returns views, clicks, and click-through rate per profile, sorted by views descending. Profiles where analytics are unavailable have null values. Call get_workspaces_for_user first to obtain the workspaceUuid.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Alias for workspaceUuid. Pass either workspaceId or workspaceUuid. |
| `workspaceUuid` | string | no | The UUID of the workspace to fetch analytics for. |

### `get_workspace_members`

Retrieve the members of a workspace. Returns workspace-user ids plus user ids, uuids, name, profile picture, and role for each member, plus pageInfo. Returns one page at a time.

- **Access:** read · **Approval:** none · **Identity:** user

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | number | no | Number of results per page (default 50). |
| `page` | number | no | Page number, 1-indexed (default 1). |
| `workspaceId` | string | no | Optional workspace ID. Defaults to the current workspace. |
| `workspaceUuid` | string | no | Optional workspace UUID alias for workspaceId. |

### `get_links`

Retrieve the ordered links on a Linktree profile. When ids are omitted, returns the full ordered links list and shop products. When a link has linkapp-specific metadata (e.g. Maps place data, YouTube/Spotify embed settings, Commerce product IDs, Chatbot external URLs), it is returned under a "linkapp" field. Oversize data blobs are sliced to the first 6KB and flagged with "truncated": true. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `ids` | number[] | no | Optional list of link IDs to retrieve. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `add_link`

Add a new URL-backed link to a Linktree profile. Supports optional title, link type, active state, target position, and duplicate suppression. The result may include needsLinkTypeChoice: true; if so, present its options to the creator and call add_link again with their selected linkType. It may include needsSetup: true; if so, relay the warning and editUrl so the creator can finish setup in the editor. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`. After a successful update, if putting their Linktree URL in a social bio is relevant to the creator's stated goal, get_profile_sharing_guidance can return the correct profile URL and placement steps; pass the same explicit username, profileUrl, query, or workspaceUuid target as the update, even if another profile is active or default. Do not repeat the suggestion if already completed or declined.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | yes | The URL to add. |
| `active` | boolean | no | Whether the link should be active immediately. Defaults to true. |
| `ifNotExists` | boolean | no | If true, returns the existing link instead of creating a duplicate. |
| `linkType` | string | no | Optional Linktree link type override. In LinkCoach v2, if multiple types match, present the options and re-call with the creator’s choice. |
| `position` | number | no | Optional target position for the link. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `title` | string | no | Optional title override. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- Calendly Embed: Optional linkType values can request paid embedded Link Apps.
- Embed LinkApp: Optional linkType values can request paid embedded Link Apps.
- Link Lock - Passcode: Optional linkType values can request paid locked links.
- Link Lock - Password: Optional linkType values can request paid locked links.
- Redirect: Optional linkType values can request paid redirect links.
- Schedule: Optional linkType values can request paid scheduled links.

### `create_collection`

Create a new empty collection on a Linktree profile. Use add_links_to_collection to put links inside it. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | The title for the new collection. |
| `position` | number | no | Optional position for the collection in the list. Defaults to 0 (top). |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `add_links_to_collection`

Move existing top-level links into a collection on a Linktree profile (which may be newly created or already exist). Call get_links first to find the collection ID and link IDs. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `collectionId` | number | yes | The numeric ID of the collection to add links to. |
| `linkIds` | number[] | yes | IDs of existing top-level links to move into the collection. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `update_link`

Update an existing link or collection on a Linktree profile. Works on top-level links, links inside a collection, and the collection itself. Supports sparse updates to title, URL, and active state; collections do not support URL updates. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`. After a successful update, if putting their Linktree URL in a social bio is relevant to the creator's stated goal, get_profile_sharing_guidance can return the correct profile URL and placement steps; pass the same explicit username, profileUrl, query, or workspaceUuid target as the update, even if another profile is active or default. Do not repeat the suggestion if already completed or declined.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | number | yes | The numeric ID of the link or collection to update. |
| `active` | boolean | no | Optional active state for the link. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `title` | string | no | Optional new title for the link. |
| `url` | string | no | Optional new URL for the link. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `reorder_link`

Reorder a top-level link, collection, or collection item on a Linktree profile. Provide exactly one of to, up, or down. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | number | yes | The numeric ID of the link to reorder. |
| `down` | number | no | Reorder the link down by N positions. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `to` | number | no | Reorder the link to an absolute position. |
| `up` | number | no | Reorder the link up by N positions. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `delete_link`

Delete a link from a Linktree profile, including a link inside a collection. Collections and headers are not deleted by this tool. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** destructive · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | number | yes | The numeric ID of the link to delete. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_link_analytics`

Retrieve analytics for a specific link on a Linktree profile, including clicks, views, CTR, and trend data. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `linkId` | number | yes | The link ID to get analytics for. |
| `endDate` | string | no | Optional end date in YYYY-MM-DD format. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `startDate` | string | no | Optional start date in YYYY-MM-DD format. |
| `timezone` | string | no | Optional timezone override for the analytics data. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

**Free-plan limitations** (some options require a paid plan):

- Individual Link Analytics: The tool retrieves individual link analytics.
- View Custom Date Ranges: Free accounts are limited to the SDK free-plan analytics date range.

### `get_social_links`

Retrieve a Linktree profile's social icon links and their display order. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `upsert_social_link`

Create or update a social icon link on a Linktree profile for a supported platform. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | yes | The social platform type, for example INSTAGRAM or YOUTUBE. |
| `url` | string | yes | The destination URL or platform value for the social icon. |
| `active` | boolean | no | Whether the social icon should be active. Defaults to true. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `reorder_social_links`

Move a social icon link to a new position on a Linktree profile. Provide exactly one of to, up, or down. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | yes | The social platform type to move. |
| `down` | number | no | Move the social icon down by N positions. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `to` | number | no | Move the social icon to an absolute position. |
| `up` | number | no | Move the social icon up by N positions. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `delete_social_link`

Delete a social icon link from a Linktree profile by platform type. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** destructive · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | yes | The social platform type to remove, for example INSTAGRAM or YOUTUBE. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_conversations`

List the conversations in a Linktree profile's messages inbox, most recent first. Use unreadOnly to find new messages, then call get_messages for each conversation. Reading does not mark conversations as read. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `cursor` | string | no | nextCursor from a previous get_conversations call, to fetch the next page. |
| `limit` | number | no | Conversations to return. Defaults to 20, max 30. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `unreadOnly` | boolean | no | Only return conversations with unread messages. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `get_messages`

Read the most recent messages in one conversation, oldest to newest. Message text is the follower's raw text. Pass nextCursor as before to read older messages. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** read · **Approval:** none · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `conversationId` | string | yes | The conversationId returned by get_conversations. |
| `before` | string | no | nextCursor from a previous get_messages call, to fetch older messages. |
| `limit` | number | no | Messages to return. Defaults to 100, max 100. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

### `send_message`

Send a reply as the creator in an existing conversation. Calling this sends the message immediately, so draft the reply with the user first. Only works in conversations where the follower has already messaged the creator. If the caller has multiple accessible profiles and no active profile is set, this returns a PROFILE_REQUIRED response with candidates. Call `set_active_profile` once, or pass `username`/`profileUrl`.

- **Access:** write · **Approval:** required · **Identity:** user_or_account

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `conversationId` | string | yes | The conversationId returned by get_conversations. |
| `text` | string | yes | The reply text to send, exactly as it should appear. |
| `profileUrl` | string | no | Optional Linktree profile URL to target, such as https://linktr.ee/example. Preferred over free-text query for multi-profile users. |
| `query` | string | no | Optional search query for discovery or disambiguation. Matches username, display name, or real name. |
| `username` | string | no | Optional Linktree username to target, with or without a leading @. Preferred selector for multi-profile users. |
| `workspaceUuid` | string | no | Optional workspace UUID to narrow discovery before resolving the target profile. |

## Error contract

Errors are returned as an MCP tool result with `isError: true` and a single text content block containing JSON: `{ "error": "<CODE>", "message": "<human-readable>" }`. Some codes carry extra fields (e.g. PROFILE_REQUIRED adds `candidates` and `nextAction`).

| Code | Meaning |
| --- | --- |
| `PROFILE_REQUIRED` | The caller has multiple accessible profiles (or none selected) and must choose one. The envelope includes `candidates` and a `nextAction`. |
| `NOT_FOUND` | A referenced resource (link, collection, integration, account) does not exist. |
| `INVALID_INPUT` | A parameter was missing, empty, or invalid for the requested operation. |
| `UNAUTHORIZED` | The request is missing required identity/authorization headers or the token cannot act on the target. |
| `RATE_LIMITED` | The caller exceeded the public MCP rate limit. Retry after backing off. |
| `UPGRADE_REQUIRED` | The tool or requested option requires a paid Linktree plan. |
| `PRODUCT_TITLE_READONLY` | The link displays the title of its linked Shop product, which link mutations cannot write. Rename the product in Shop instead. Nothing was changed. |
| `FEATURED_LAYOUT_INELIGIBLE` | The link does not meet the featured layout prerequisites: a non-SVG image thumbnail that is not an icon, and no animation. Nothing was changed. |
| `IMAGE_IMPORT_FAILED` | The image URL could not be copied into Linktree-hosted storage, so it was not used. Nothing was changed. Use a different, publicly reachable image URL. |
| `UPSTREAM_ERROR` | A downstream Linktree service returned an error while handling the request. |
| `UNEXPECTED_ERROR` | An unclassified error occurred. Safe to retry once; report if it persists. |

Example `PROFILE_REQUIRED` response body (inside the text content block):

```json
{
  "error": "PROFILE_REQUIRED",
  "reason": "multiple",
  "message": "Multiple accessible Linktree profiles are available. Call set_active_profile with a username to choose one for this conversation, or pass username/profileUrl on this call.",
  "candidates": [
    {
      "username": "creator-one",
      "displayName": "Creator One",
      "profileUrl": "https://linktr.ee/creator-one",
      "accountUuid": "acc-1",
      "isSelectedAccount": false,
      "isEditable": true
    }
  ],
  "nextAction": {
    "hint": "Call set_active_profile with { username } to stick this profile for the rest of the conversation, or pass username/profileUrl on each call.",
    "tool": "set_active_profile"
  }
}
```

