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

## BLOCKED - Critical Bugs Preventing Implementation

**Status:** 2 blocking issues prevent 5000-song handling. WS4 Step 2 diagnosis complete (2026-05-08). Ready for debugging session.

### Bug #1: 429 Rate Limit Hang (Last occurred: 2026-03-28)

**Symptom:** Script processes ~24 tracks successfully, then hangs indefinitely when Spotify API returns 429 (rate limit).

**Root cause identified:**
- urllib3 has 429 in `status_forcelist` (automatically retries silently)
- `_retry_call()` sleeps exponentially (can exceed 60s)
- No timeout cap on retry sleep
- SEARCH_DELAY_S=0.1 is too aggressive for large libraries (5471 tracks)

**Fixes to apply:**
1. Remove 429 from urllib3's status_forcelist (explicit handling instead)
2. Cap retry sleep to 30s maximum
3. Increase SEARCH_DELAY_S from 0.1 to 0.5 (or adaptive backoff)
4. Add heartbeat logging (print track name BEFORE API call)

**Test:** Dry-run on sample 500-track subset, monitor for 429 responses

### Bug #2: Unicode Artist Matching Fails (Observed: 2026-03-28 log line 52)

**Symptom:** "JAŸ-Z" (with diacritic) matches as LOW confidence instead of EXACT. Fails to find legitimate tracks.

**Example from log:**
```
[6/2597] Jay-Z - Encore
- LOW (partial: JAŸ-Z - Numb / Encore)
```

**Root cause:** `normalise()` function doesn't strip diacritics. "JAŸ-Z" != "JAY-Z".

**Fix to apply:**
- Add NFKD normalization to `normalise()` to decompose diacritics: `unicodedata.normalize('NFKD', s)`
- Test cases: JAŸ-Z → JAY-Z, Dön → Don, etc.

**Test:** Match tracks with diacritics in artist names (French, German, Spanish artists)

---

## Next Session - Implementation Scope

**Estimated:** 2 sessions (both bugs fit in one session - Unicode fix is 2 lines)

**Session 1: Fix Both Bugs**
- [ ] Locate urllib3 retry config in spotipy wrapper
- [ ] Remove 429 from forcelist
- [ ] Cap _retry_call sleep to 30s
- [ ] Increase SEARCH_DELAY_S to 0.5
- [ ] Add heartbeat logging (print track name BEFORE API call)
- [ ] Add NFKD normalization to `normalise()` in matcher.py: `unicodedata.normalize('NFKD', s)`
- [ ] Add test cases for diacritics (JAY-Z with diacritic -> JAY-Z, etc.)
- [ ] Commit: "fix: 429 rate limit hang + Unicode normalization in artist matching"

**Session 2: Full Validation**
- [ ] Run on full 5471-track library
- [ ] Monitor for hangs + match rate improvement
- [ ] Review data/reports/match_report.txt
- [ ] Commit: "test: Full library sync validation"

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
- [ ] Rename "NONE" to "NOT FOUND" - clearer CLI output
- [ ] Stricter types - Pylance reports 900+ errors, add type hints gradually
- [ ] Rename project - consider "MusicLibPlaylistSyncer" across files/imports/READMEs
- [x] Terminal output: each track on its own line (fixed in 186be2b)
- [ ] Log file results section is huge, format nicer for humans
- [ ] `--reset-exhausted` CLI flag - retry all exhausted tracks after search logic improvements
- [ ] Run AudioManager before running this program so Audio Mirror is up to date
- [ ] diagnose.py = integrate into main program as one liner in terminal output, running diagnostic checks!
