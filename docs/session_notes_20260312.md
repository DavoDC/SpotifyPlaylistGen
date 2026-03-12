# Session Notes 2026-03-12 (Session 2)

## Run Results (21:31 run)
- 5471 tracks in library, 1103 already synced, 4368 to search
- Got to track 649/4368 before hanging (~15%)
- 500 tracks successfully uploaded in 5 batches of 100
- Match rate ~78% (500 matched / 637 searched) — MUCH better than before
- No report generated (run didn't complete)
- 22 instances of HTTP 429 in log

## Critical Issue: 429 Rate Limit Hang
- Log ends at track 649 with HTTP 429 on search
- spotipy's urllib3 Retry has 429 in `status_forcelist` — it auto-retries with Retry-After header
- Retry-After can be huge (minutes/hours), causing program to freeze
- Our `_retry_call` wrapper ALSO handles 429 — double retry!
- **Fix needed**: Remove 429 from `status_forcelist`, let `_retry_call` handle it with capped sleep + logging
- Also increase `SEARCH_DELAY_S` from 0.1 to 0.5 to reduce rate limit hits

## Critical Issue: JAY-Z Unicode Bug
- Spotify returns artist as `JAŸ-Z` (with `Ÿ` = U+0178) not `JAY-Z`
- `score_match` compares `jay-z` vs `jaÿ-z` → NONE for ALL Jay-Z tracks
- Same issue likely affects other artists with special chars
- `normalise()` needs to strip diacritics/accents (unicodedata.normalize NFKD + strip combining chars)
- This explains Jay-Z Song Cry, Renegade, Young Forever, Empire State of Mind ALL returning NONE despite being found by search

## Other NONE Patterns Observed
- Joey Bada$$ — `$` in name may cause search/match issues
- Justin Bieber collab tracks (2U, Cold Water, Let Me Love You) — primary artist in XML is Bieber but on Spotify it's the other artist (David Guetta, Major Lazer, DJ Snake)
- Kanye VULTURES album tracks — may genuinely not be on Spotify
- Jersey Boys — likely soundtrack not on Spotify

## What Was Fixed This Session (before run)
- `setup_logging()` split: DEBUG to file, INFO to terminal
- All per-track logs ([SEARCH], [MATCH], [RESULT], [DECISION]) → logging.debug()
- Added [API] debug logging to `_retry_call`
- Terminal output: each track on its own line (no more \r overwrite)
- TODO.md merged into docs/SpotifyPlaylistGen.md

## Commits (unpushed)
- 203b58e - Split log levels: DEBUG for per-track detail, INFO for progress
- 4ca4ec7 - Merge TODO.md into docs/SpotifyPlaylistGen.md
- 186be2b - Fix terminal output: each track result on its own line

## Next Session Priority
1. Fix 429 hang: remove from status_forcelist, cap retry sleep, increase SEARCH_DELAY_S
2. Fix unicode artist matching (JAŸ-Z bug): add NFKD normalization
3. Re-run and verify
