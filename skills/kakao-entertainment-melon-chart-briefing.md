---
name: melon-chart-briefing
description: Read the official Melon music charts and turn them into a briefing, optionally with playable links, using the Melon MCP server.
api: kakao-entertainment-melon-mcp
transport: mcp
endpoint: https://mcp.melon.com/mcp
operations:
  - get_music_chart
  - get_music_content_details
  - get_artist_contents
  - create_playback_url
---

# Melon chart briefing

Every tool named here was read live from the Melon MCP server's `tools/list` on 2026-08-23. Do not call
a tool that is not in this list — the server's tool set has already drifted from its launch
documentation, so names taken from the blog guide may not exist.

## Before you start

- The endpoint is `https://mcp.melon.com/mcp`, Streamable HTTP only. SSE will not connect.
- Authentication is OAuth 2.0 against `https://cola.melon.com` with PKCE (S256). Your redirect URI must
  already be on Melon's per-partner allowlist; if you get `invalid_redirect_uri`, registration is the
  problem, not your code.
- `tools/list` answers anonymously. Tool invocation does not.
- The server is beta and the provider offers it "as is".

## Steps

1. **Pick the chart.** Call `get_music_chart`. `chart_type` is required and must be one of `TOP_100`
   (realtime), `DAILY`, `WEEKLY`, `HOT_TRACK_LIKE`, `HOT_TRACK_SEARCH`, `HOT_TRACK_DJ`, or `HOT_100`.
   For `HOT_100`, `hot_100_filter` selects the release window. Paginate with `start` (1-based, minimum
   1) and `size` (maximum 30 for this tool — it is lower than the 100 other tools allow, so do not
   assume a shared page size).
2. **Deepen the entries you will actually talk about.** Pass the song, album or artist IDs from step 1
   to `get_music_content_details` — `content_type` and `content_ids` are both required, and the tool
   accepts multiple IDs in one call, so batch rather than looping.
3. **Add artist context only if asked.** `get_artist_contents` takes `artist_id` and returns that
   artist's songs or albums; `participation_type` distinguishes their own releases from features.
4. **Make it playable if the user wants to listen.** `create_playback_url` composes a melon.com URL from
   `song_ids`, `playlist_seqs`, `album_ids` or `mv_id`. It is annotated read-only and non-destructive by
   the provider — it generates a link, it does not start playback on the user's account. Nothing here
   changes state, so nothing here needs undoing.

## Conventions to respect

- This server uses two pagination vocabularies. Charts and likes use `start`/`size`; search, artist,
  genre, playlist and history tools use `page`/`limit` with `limit` capped at 100. Read each tool's
  `inputSchema` rather than assuming one convention.
- 16 of the 18 tools declare an `outputSchema`. Type the response from it rather than pattern-matching
  the text. `get_genres` and `get_song_streaming_report` are the two that do not.
- The catalogue is predominantly Korean. Search accepts Korean keywords; `ui_locales_supported` on the
  authorization server is `ko` only.

## Errors

- **Content Not Found** — the ID or query did not resolve. Re-derive the ID from search or chart output
  rather than constructing one.
- **Permission Denied** — the account lacks the required subscription state. Chart and search work
  without one; personalization does not.
- Regional licensing can make individual tracks unavailable. `get_playlist_tracks` exposes
  `service_available` for exactly this; filter on it before promising a user something is playable.

## Do not

- Do not invent a rate limit or a retry budget. None is published and the server returns no
  `RateLimit-*` or `Retry-After` headers, so back off conservatively on any failure.
- Do not treat the guide's tool names as authoritative. `search_melon_magazines`, `get_artist_songs`,
  `get_main_genres`, `get_my_followed_artists`, `get_my_song_streaming_history` and
  `get_song_streaming_stats` appear in the guide but are **not** on the live server.
