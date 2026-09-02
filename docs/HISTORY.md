# SpotifyTools - Change History

## 2026-03-24 / 2026-03-25

**429 rate limit hang fixed** (`a4fe638`) - Root cause: urllib3 v2 was internally sleeping for uncapped Retry-After values even when 429 was not in `status_forcelist`, bypassing `_retry_call` entirely. Fix: `retries=0` on `spotipy.Spotify` - all retry logic now in `_retry_call` with a 30s cap. `SEARCH_DELAY_S` increased from 0.1 to 0.5.

**Unicode diacritic matching fixed** - `normalise()` in `matcher.py` now applies NFKD decomposition before stripping combining characters, so "JAŸ-Z" matches "JAY-Z" as EXACT/HIGH instead of LOW. Test coverage: 5 diacritic-specific tests added to `test_matcher.py`.

---

## 2026-05-25

**Sequential open_playlist flow** - Replaced the fire-and-forget all-tracks-at-once loop in `open_playlist.py` with a one-at-a-time interactive flow. Each track now shows `[i/N] Artist - Title`, opens in the local music manager, then waits for Enter before opening the next. The confirm prompt previews the upcoming track. Supports 'q' to quit mid-session and Ctrl+C for clean exit. Previously all N tracks opened simultaneously into browser tabs making the tool unusable for large playlists.
