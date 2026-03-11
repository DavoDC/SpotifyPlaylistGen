# SpotifyPlaylistGen — Strategic Recovery Plan
==============================================

## 🔴 PRIORITY 1: The "NONE" Match Crisis (Bug Investigation)
*Current State: Popular tracks (Ariana Grande, Calvin Harris) return NONE despite existing on Spotify.*

- [ ] **Debug Search Queries:** Log the EXACT string sent to `sp.search()`. Check if we are including unnecessary XML metadata (e.g., file paths, bitrates, or "Remastered" tags) that breaks Spotify's search.
- [ ] **Loosen Matcher Logic:** Review `src/matcher.py`. If a track is 90% similar but failing "EXACT," ensure it falls into "HIGH" rather than dropping to "NONE."
- [ ] **Query Fallback Strategy:** If `artist + title + album` returns 0 results, retry with just `artist + title`.
- [ ] **Artist Name Normalization:** Handle "feat.", "&", and "and" in artist fields to ensure the search string is clean.

## 🟠 PRIORITY 2: Observability & "Claude-Friendly" Logs
*Goal: Ensure the AI and User know exactly what the script is doing at every second.*

- [ ] **CLI Transparency:** Rename "NONE" status to "NOT FOUND" or "SEARCH FAILED" for clarity.
- [ ] **Real-time Heartbeat:** Ensure the CLI prints the current track being processed *before* the API call starts (prevents the "stuck" feeling).
- [ ] **Enhanced Logging:** Implement `--verbose` or update `logs/` to include:
    - Full JSON response from Spotify for failed searches.
    - The "Confidence Score" breakdown for why a match was rejected.
    - Snapshot ID after every playlist modification.

## 🟡 PRIORITY 3: Live Verification & Polish
- [ ] **Rename Project:** Rename to `MusicLibPlaylistSyncer` across all files, imports, and READMEs.
- [ ] **First Real-World Run:** Run via `scripts/run.bat` (no simulation) and monitor the first 20 tracks.
- [ ] **Report Review:** Audit `data/reports/` to ensure the "LOW confidence" tracks are formatted for easy manual reading/copy-pasting.

## 🟢 FUTURE/STRETCH
- [ ] **Rate Limit Handling:** Verify exponential backoff if running against very large libraries (1000+ tracks).
- [ ] **History Cleanup:** Add a command to "Reset Exhausted" tracks to retry them after the search logic is fixed.