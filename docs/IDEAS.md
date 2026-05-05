## TODO

NEED TO FIX THIS UP, CONSOLIDATE ALL DOCS ETC

### TIER 2 - Future Ideas (needs investigation)

- **SpotifyTools: generalize as reusable Spotify layer for other programs**
  - AudioManager is a C# program - a Python module won't work directly. Would require IPC (inter-process communication), a daemon-to-daemon design (SPG exposes a local HTTP server?), or a named pipe approach.
  - This needs more investigation and a clear use-case before starting. What does AudioManager actually need from Spotify? Browse-by-playlist? Real-time search? The answer changes the architecture significantly.
  - Python module packaging makes sense ONLY if another Python program is the consumer. Do not start this until there is a confirmed second consumer with a defined interface need.

### Housekeeping
- [ ] Fix CLAUDE.md - replace all em dashes (--) with regular hyphens. Guard hook blocks edits to the file until this is done. Rewrite the whole file in one Write call.

### Next
- [ ] Apply Critical Fix 1 — 429 Rate Limit Hang: Remove 429 from status_forcelist so urllib3 doesn't silently block. Cap _retry_call's 429 sleep to 30s. Increase SEARCH_DELAY_S from 0.1 to 0.5 (already partially applied)
- [ ] Apply Critical Fix 2 — Unicode Bug: Add NFKD normalizatBoth preserve ion to normalise() to strip diacritics (JAŸ-Z → JAY-Z).
- [ ] Primary Goal is to handle 5000 songs in one script run without hanging!
- [ ] Fix 429 hang: remove from `status_forcelist`, cap retry sleep, increase `SEARCH_DELAY_S`
- [ ] Fix unicode artist matching (JAŸ-Z bug): add NFKD normalization to `normalise()`
- [ ] Re-run and verify match rate + completion
- [ ] Review match report in `data/reports/` — confirm match rate improved

### Polish

- [ ] Optimize logs for Claude token usage, reduce size
- [ ] Rename "NONE" to "NOT FOUND" — clearer CLI output
- [ ] Heartbeat — print track name BEFORE API call starts (prevents "stuck" feeling)
- [ ] Stricter types — Pylance reports 900+ errors, add type hints gradually
- [ ] Rename project — consider "MusicLibPlaylistSyncer" across files/imports/READMEs
- [x] Terminal output: each track on its own line (fixed in 186be2b)
- [ ] Log file results section is huge, format nicer for humans
- [ ] `--reset-exhausted` CLI flag — retry all exhausted tracks after search logic improvements
- [ ] Run AudioManager before running this program so Audio Mirror is up to date
- [ ] diagnose.py = integarte into main program as one liner in terminal oputput, runnign diagnostic checks!