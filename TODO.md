# SpotifyPlaylistGen — TODO

## DONE (2026-03-12 stabilization)

- [x] **_wait_for_exit() recursion crash** — fixed, was calling itself instead of input()
- [x] **"NONE" match crisis** — score_match() now cleans BOTH local and Spotify titles; enhanced clean_title() strips metadata tags
- [x] **Premature "added" state** — history now only marks "added" after Spotify confirms upload
- [x] **LOW confidence exhaustion gap** — LOW matches now properly reach "exhausted" after 5 attempts
- [x] **search_attempts off-by-one** — now increments correctly on exhausted transition
- [x] **Query fallback** — already implemented (strict search then broad fallback)

## Priority 1: Live Verification

- [ ] **Run live via `scripts/run.bat`** — verify the matcher fixes resolve the NONE crisis for the 4868 unmatched tracks
- [ ] **Reset exhausted tracks** — tracks incorrectly exhausted under old broken matcher should be reset to retry with fixed logic. Add a `--reset-exhausted` CLI flag or one-time script
- [ ] **Review match report** — check `data/reports/` after live run to confirm match rate improved

## Priority 2: Observability

- [ ] **Rename "NONE" to "NOT FOUND"** — clearer CLI output
- [ ] **--verbose flag** — log full Spotify JSON response for failed searches, confidence score breakdown
- [ ] **Heartbeat** — print track name BEFORE API call starts (prevents "stuck" feeling)

## Priority 3: Polish

- [ ] **Stricter types** — Pylance reports 900+ errors, add type hints gradually
- [ ] **JS files use require()** — convert to ES module imports (read-pdf.js etc. in workspace tools, not this repo)
- [ ] **Rename project** — consider "MusicLibPlaylistSyncer" across files/imports/READMEs
- [ ] **Artist name normalization** — handle "feat.", "&", "and" in artist search queries (partially addressed by clean_title but artist field not yet cleaned)

## Stretch

- [ ] **Rate limit verification** — confirm exponential backoff works for 5000+ track runs
- [ ] **History cleanup command** — `--reset-exhausted` to retry all exhausted tracks after search logic improvements
