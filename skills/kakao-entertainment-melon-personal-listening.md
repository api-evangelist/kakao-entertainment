---
name: melon-personal-listening
description: Answer questions about a signed-in Melon member's own listening history and taste, and recommend from it, using the Melon MCP server.
api: kakao-entertainment-melon-mcp
transport: mcp
endpoint: https://mcp.melon.com/mcp
operations:
  - get_my_most_listened_songs
  - get_recently_played_music_contents
  - get_my_liked_music_contents
  - get_my_created_playlists
  - get_playlist_tracks
  - get_song_streaming_report
  - recommend_personalized_songs_by_dj_mallang
  - recommend_similar_songs_by_dj_mallang
---

# Melon personal listening

These tools read the authenticated member's own data. Every tool below was read live from `tools/list`
on 2026-08-23 and is annotated `readOnlyHint: true` by the provider — nothing in this skill mutates the
user's account, and nothing in it needs an undo.

## Before you start

- Requires an OAuth 2.0 access token from `https://cola.melon.com`. The scopes the authorization server
  advertises are `melonAuthentication`, `melonAccessAuthority`, `melonService`, `memberKey`,
  `memberName`, `memberNickname`, `ipinGender`, `ipinBirthDate`, `realNameYn` and `ticketService`. Melon
  publishes no mapping from scope to tool, so request the minimum you can and widen only on a
  Permission Denied.
- Several of these scopes carry identity and Korean i-PIN verification data (gender, birth date,
  real-name status). Do not request them to answer a music question.
- The provider states personalization features require an active Melon subscription.

## Steps

1. **Establish taste before recommending.** `get_my_most_listened_songs` takes `date_type` and
   `term_list` and is the tool for "what did I play most last month / this time last year".
   `get_recently_played_music_contents` covers the short window. `get_my_liked_music_contents` takes a
   `content_type` and returns liked songs, albums and playlists.
2. **Recommend from it.** `recommend_personalized_songs_by_dj_mallang` takes only `size` and uses
   Melon's own DJ Mallang engine over the member's history, likes and playlists.
   `recommend_similar_songs_by_dj_mallang` takes a single `song_id` and returns up to 30 stylistically
   similar tracks — use this one when the user points at a specific song rather than asking generally.
3. **Work with their playlists.** `get_my_created_playlists` lists what the member has built;
   `get_playlist_tracks` expands one by `playlist_seq`. Pass `service_available` when you need to know
   which tracks are actually streamable in the user's region.
4. **Answer play-count questions precisely.** `get_song_streaming_report` requires `song_id` and
   `report_type`: `public` returns the song's overall Melon streaming count and listener count, `my`
   returns the member's own history for that song. Choose deliberately — these are different claims.

## Conventions to respect

- Pagination is `page`/`limit` for most of these tools (`limit` maximum 100) but `start`/`size` for
  `get_my_liked_music_contents`, and `recommend_personalized_songs_by_dj_mallang` takes only `size`.
- `get_song_streaming_report` is one of only two tools with no declared `outputSchema`; parse its result
  defensively.

## Privacy

This skill reads a real person's listening history. Report it back to that user and no further. Do not
persist it, do not aggregate it across sessions, and do not use identity scopes to profile the member.
The provider states it does not store authentication tokens on the MCP server; hold yours to the same
standard.

## Do not

- Do not claim you have added, removed or reordered anything in the member's library. The live server
  ships no playlist-mutation tool, even though the provider's overview prose mentions managing
  collections. If the user asks you to change a playlist, say plainly that the public MCP surface is
  read-only and point them at the Melon app.
