# SpotifyPlaylistGen - Change History

## 2026-05-25

**Sequential open_playlist flow** - Replaced the fire-and-forget all-tracks-at-once loop in `open_playlist.py` with a one-at-a-time interactive flow. Each track now shows `[i/N] Artist - Title`, opens in the local music manager, then waits for Enter before opening the next. The confirm prompt previews the upcoming track. Supports 'q' to quit mid-session and Ctrl+C for clean exit. Previously all N tracks opened simultaneously into browser tabs making the tool unusable for large playlists.
