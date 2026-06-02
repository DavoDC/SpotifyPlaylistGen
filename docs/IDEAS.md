# Spotify Playlist Generator - Ideas and TODOs

**Goal:** Standalone tool that reads an offline music library (AudioMirror XML) and creates a matching Spotify playlist, so David can listen to his library at work where only Spotify is available.

## Architecture Notes

- Read AudioMirror XML (output of AudioManager) - structured metadata, faster than reading MP3s directly. XML has: Title, Artists (semicolon-separated), Album, Year - no ISRC.
- Search Spotify using primary artist (first before `;`) + title + album
- MusicBrainz only as fallback if match quality is poor
- Show results with match confidence - user can review/adjust before committing
- Spotify auth: Authorization Code + PKCE flow, scopes `playlist-modify-private`, `playlist-read-private`, playlists created **private** by default
- Featured artists: strip `feat.`/`;` variants before searching; handle remix suffixes, alternate titles
- **Standalone first** - build standalone before considering DWave/AudioManager integration

---

## Full Library Validation (needs credentials + AudioMirror XML)

Both blocking bugs were fixed 2026-03-24/25 (urllib3 retries=0, SEARCH_DELAY_S=0.5, NFKD normalization). 190 tests pass. The tool is ready for a real run on the full library.

- [ ] Run on full 5471-track library with real credentials
- [ ] Monitor for hangs + check match rate improvement
- [ ] Review `data/reports/match_report.txt`
- [ ] Commit: "test: Full library sync validation"

Note: requires the AudioMirror XML and Spotify credentials on the home PC (Raphael).

---

## TODO

### Repo Scope

Addressed 2026-06-03: CLAUDE.md now explains both tools (main.py sync + open_playlist.py browser) and has correct project structure. README.md is complete.

Remaining (optional):
- [ ] **docs/TOOLS.md** - detailed per-tool reference: parameters, input/output, internals. Not blocking - CLAUDE.md + README cover the basics for now.
- Should `open_playlist.py` be its own repo? Current answer: no. It's small, shares the Spotify client, and has no other consumers. Revisit if it grows significantly.

### TIER 2 - Future Ideas (needs investigation)

- **SpotifyTools: generalize as reusable Spotify layer** - DEFERRED indefinitely. No second Python consumer exists. AudioManager is C# - any integration would require IPC with an undefined interface. The internal `spotify_client.py` is the right structure. Revisit only if a second repo needs Spotify access and has a defined interface requirement.

### Housekeeping
- [x] Fix CLAUDE.md em dashes - done 2026-06-03. Updated project structure and added two-tool scope note at same time.

### Polish

- [ ] Optimize logs for Claude token usage, reduce size
- [x] Rename "NONE" to "NOT FOUND" in CLI display - done 2026-06-03
- [ ] Stricter types - Pylance reports 900+ errors, add type hints gradually
- [ ] Rename project - consider "MusicLibPlaylistSyncer" across files/imports/READMEs
- [x] Terminal output: each track on its own line (fixed in 186be2b)
- [ ] Log file results section is huge, format nicer for humans
- [x] `--reset-exhausted` CLI flag - done 2026-06-03. Resets all exhausted tracks to unmatched so they get retried next run.
- [ ] Run AudioManager before running this program so Audio Mirror is up to date
- [x] diagnose.py hint on auth failure - done 2026-06-03. Auth error now suggests running `python src/diagnose.py`.
