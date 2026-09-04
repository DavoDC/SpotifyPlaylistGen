# SpotifyTools - Ideas and TODOs

## ACTIVE PLAN: Rename to SpotifyTools + fix import fragility (2026-09-02)

Scope has grown beyond "playlist gen": this repo is now a Spotify API client + AudioMirror-XML sync engine + interactive playlist browser + acquire-flow orchestration consumed directly by AudioManager's Python GUI (`gui/tabs/acquire.py`, in-process `sys.path.insert` sibling import, not IPC - see AudioManager/docs/References/GUI-Architecture.md "Resolved 2026-08-31": deliberately no separate shared-library repo, revisit only if a second real consumer appears).

Do ONE task at a time, verify before moving to the next. Do NOT batch.

- [x] **Task 1 - package rename (do first, lowest risk):** rename `src/` -> `spotify_tools/` in this repo. Update AudioManager's ~7 `from src.X import Y` lines (`gui/tabs/acquire.py`, `gui/config.py`, `gui/tests/test_acquire.py`) to `from spotify_tools.X import Y`. Run AudioManager's `test_acquire.py` to verify green. Commit both repos separately. Done 2026-09-02: git mv preserved history; SpotifyPlaylistGen commits f714476 (rename) + 98f48df (import/doc updates, 214 tests pass); AudioManager only had import sites in `gui/tabs/acquire.py` (8, not gui/config.py or test_acquire.py) - commit dd3f4a73, 17 tests pass.
- [x] **Task 2 - path-resolution helper:** replace the hardcoded `SPOTIFYGEN_ROOT = REPO_ROOT.parent / "SpotifyPlaylistGen"` in AudioManager's `gui/config.py` and the hardcoded absolute `config.json` path in `src/config.py`'s `CONFIG_PATH` with one helper that checks a `SPOTIFY_TOOLS_ROOT` env var first, falls back to the sibling-folder assumption. Single point of failure instead of duplicated hardcoding. No packaging/pyproject.toml - not needed, no external consumer. SpotifyTools side done 2026-09-04, alongside the BUG item below - see that entry for what was actually built (`src/spotify_tools/paths.py`) and why per-file dirname patching wasn't enough on its own. **Correction, same day:** this had been checked off as fully done, but the AudioManager-side half (`gui/config.py`'s `SPOTIFYGEN_ROOT`) was never actually touched - still a bare hardcode, confirmed via `grep` while doing the boundary/docs audit and via this repo's own line 27 above ("Task 2's env-var helper is still separate/undone"). Fixed for real 2026-09-04: `gui/config.py`'s `SPOTIFYGEN_ROOT` now checks the SAME `SPOTIFY_TOOLS_ROOT` env var (falls back to the sibling-folder assumption), so one variable controls both sides of the `sys.path.insert` boundary. Regression tests: `gui/tests/test_config.py` (2 tests). AudioManager suite: 124 passed.
- [x] **Task 3 - repo/folder rename to SpotifyTools (do on a clean tree, not mid-feature-edit):** rename the `SpotifyPlaylistGen` folder/repo to `SpotifyTools`. Checklist (blast radius found during planning, verify each):
  - GitHub remote name / `gh repo rename` (or manual remote update)
  - AudioManager `SPOTIFYGEN_ROOT` (or the Task-2 env var default) -> new folder name
  - Any personal notes/indexes that reference the old repo name (check separately, not tracked here)
  - Local dev-tool settings in both repos with absolute-path permissions referencing the old path
  - Desktop shortcuts / launcher scripts referencing the old folder name
  - `scripts/run.bat` / `run.sh` if they hardcode the folder name
  - README.md / CLAUDE.md self-references, badges
  - Sync the rename to any other machine this repo is checked out on - old folder won't rename itself there
  - Add one clarifying line to the new README.md/CLAUDE.md: consumed via direct sibling-import by AudioManager's Python GUI, not an installed/packaged library - avoids the name implying an architecture that was explicitly rejected.
  - `__pycache__`/stale bytecode cleanup after rename

  Done 2026-09-02. GitHub repo renamed `DavoDC/SpotifyPlaylistGen` -> `DavoDC/SpotifyTools` via `gh repo rename` (visibility unchanged, PUBLIC); `gh` did not auto-update the local `origin` URL, fixed manually with `git remote set-url`. Local folder rename (`Rename-Item`) initially failed with Access Denied twice - root cause was AudioManager's GUI (`pythonw gui/main.py`) running and holding a handle via its sibling-import of this repo; stopped that process (with explicit go-ahead) and the rename then succeeded, `.git` history intact. No Desktop/.lnk/.bat/.ps1 shortcuts found referencing the old path. `__pycache__` cleared in both this repo and AudioManager.

  Files changed - SpotifyTools (commit `54e7f20`): `CLAUDE.md`, `README.md` (added sibling-import clarifying note; also fixed a stale `src/` listing left over from Task 1), `docs/HISTORY.md`, `docs/References/DevContext.md`, `docs/SpotifyAPI_Reference.md`, `docs/spotifyplaylistgen.md`, `scripts/open_playlist.bat`, `scripts/run.bat`, `scripts/run.sh` (also fixed stale `src/` module paths left over from Task 1), `spotify_tools/main.py`, `spotify_tools/report_generator.py`.

  Files changed - AudioManager (commit `49576213` code, `224cc45b` docs): `gui/config.py` (`SPOTIFYGEN_ROOT` string only, per plan - Task 2's env-var helper is still separate/undone), `gui/tabs/acquire.py`, `gui/tests/test_acquire.py` (sibling `sys.path.insert` target), `docs/Development/HISTORY.md`, `docs/Development/IDEAS.md`, `docs/References/AudioMirror-Format.md`, `docs/References/GUI-Architecture.md`, `docs/References/Music-Discovery-Workflow.md`. Left `docs/Historical/WorkflowExecution-2026-04-26/STAGE_2_ACQUIRING_(DONE).md` untouched - dated post-mortem, accurate as written.

  Files changed - the personal notes/index mentioned above (repo index + dev-environment reference doc, in the private workspace tracked separately from this repo, also fixed a pre-existing stale exact test count while touching that line). Session-archive/phase/log entries there left untouched as historical record.

  Tests: SpotifyTools `python -m pytest tests/ -q` - all pass. AudioManager `python -m pytest gui/tests/test_acquire.py -q` - all pass.

  New GitHub URL: https://github.com/DavoDC/SpotifyTools

  Follow-up 2026-09-02: `spotify_tools/` had been placed directly at the repo root, flattening the standard `src/` layout used elsewhere in this workspace. Re-nested it to `src/spotify_tools/` (`git mv`, history preserved) to match the workspace convention while keeping the namespaced `from spotify_tools.X import Y` imports AudioManager needs. Added `pytest.ini` (`pythonpath = src`) so tests still resolve the package; `scripts/run.sh` and `scripts/open_playlist.bat` now `cd` into `src/` before `python -m spotify_tools.X`. AudioManager's `gui/tabs/acquire.py` and `gui/tests/test_acquire.py` sys.path inserts updated to append `/ "src"`. SpotifyTools commit `624d88c`, AudioManager commit `8736476c`. Both test suites pass (SpotifyTools 214, AudioManager's `test_acquire.py` 17).

- [x] **BUG (regression from the 624d88c re-nesting, found 2026-09-03): `CONFIG_PATH` resolves to a non-existent path** `[SONNET]` - CONFIG_PATH itself was already fixed by an earlier session (commit `2fca625`, added `tests/unit/test_config.py::test_config_path_resolves_to_a_real_file`) before this session started. While re-verifying that fix (2026-09-04), found the SAME 2-dirname off-by-one bug still live in four other files that were never touched by `2fca625`: `spotify_client.py`'s `CACHE_PATH` (confirmed actively broken - two divergent `.cache` OAuth-token files existed on disk simultaneously, `data/.cache` stale since Sep 2 and a duplicate at the wrong `src/data/.cache` actively written Sep 3-4), `main.py`'s `HISTORY_PATH`/`LOCK_PATH`/`LOG_DIR`/`REPORT_DIR` (not yet actually hit - `main.py` hadn't been run since the re-nesting, so no history-data loss occurred, but the bug was live and would have silently reset 5000+ tracks of sync history on next real run), `diagnose.py`'s `CONFIG_PATH`/`LOG_DIR`, and `open_playlist.py`'s `CACHE_DIR`/`LOG_FILE`. Root cause of why the first fix missed these: each file recomputed its own `BASE_DIR` independently from its own `__file__`, so fixing one file's dirname count did nothing for the others - exactly the fragility Task 2 (above) was proposed to eliminate. Fix: built `src/spotify_tools/paths.py` - one file computes `REPO_ROOT` (env var `SPOTIFY_TOOLS_ROOT` first, else a single dirname chain from paths.py's own location), every other file now imports `REPO_ROOT` from there instead of deriving its own. `diagnose.py` and `open_playlist.py` needed a second constant (`SRC_DIR`, kept for their `sys.path.insert`) since they use `src/` for import resolution but the repo root for data/config - the previous code conflated the two, which is part of why the bug existed. Also found and fixed a genuine safety issue while adding the regression test: `diagnose.py` had no `if __name__ == "__main__":` guard, so merely *importing* it (as the new test needed to, to check its path constants) executed a live Spotify diagnostic - real network calls, potentially a real OAuth prompt - the test caught this before it reached a real API call. Wrapped the whole diagnostic body in `_run_diagnostics()` behind the guard; behavior is unchanged for its real invocation (`python -m spotify_tools.diagnose`). Regression tests: `tests/unit/test_paths.py` (6 tests - REPO_ROOT resolves correctly, env var override works, and each of the four files' constants resolve under repo root not `src/`). Full suite: 221 passed. **Not done:** manually re-verifying AudioManager's Acquire tab against a real playlist id - needs David's real Spotify credentials, not available to this session. **Left as-is, not cleaned up:** a stray `src/data/logs/open_playlist.log` (bug-artifact from the old broken path) that a live process held a lock on at the time of cleanup - the `.cache` duplicate under the same stale `src/data/` was removed; this one file is harmless leftover, safe to delete manually once nothing has it open.

## Acquire-tab investigations (2026-09-05)

Two items raised from AudioManager's `docs/Development/IDEAS.md` ("[GUI] Acquire tab polish", 2026-09-01). Both investigated against a live Deemix and live Spotify. Suite went 221 -> 241 tests, all green.

- [x] **Per-row Deemix links "broken" - root-caused and fixed (commit `b6237dc`).** The reported symptom was real but the suspected cause was not. Repro was possible after all: Deemix runs here as a Docker container (`ghcr.io/bambanah/deemix`, bound to `127.0.0.1:6595`), so the link was tested end to end in a browser rather than only statically. Everything suspected in the original note was **correct already**: the `http://localhost:6595/search` scheme, the `?term=` parameter name, and the URL encoding. Confirmed by reading the served SPA bundle - the router registers a real `/search` route (`createWebHistory`, base `/`, so the path is directly navigable and the server catch-alls to `index.html`), and `SearchView` computes its term as `route.query.term || <stored query>` then calls `performMainSearch` during `setup()`. Navigating to a built URL in Chrome returned the correct track as the first result.

  The actual bug was in the query text, in `_FEAT_RE` in `open_playlist.py`. The pattern `\s*[\(\[]?(?:feat|ft)\.?\s+.*` had no word boundary, so `feat`/`ft` matched **inside ordinary words** and truncated the title from that point on: "Left Behind" became "Le", "Defeat Me" became "De", "Lift Me Up" became "Li", "Drift Away" became "Dri". Those rows' links then searched Deemix for nonsense, which is exactly the "broken links" symptom. Fixed by adding `\b` around the alternation. 11 tests cover both directions (words merely containing ft/feat survive; genuine "feat."/"ft." credits still stripped). Verified post-fix against the live Deemix API: "Dobie Gray Drift Away" and "Rihanna Lift Me Up" now both return the exact track as the first hit.

  Still open, lower priority, no evidence it has bitten yet:
  - `MANAGER_URL` is hardcoded to `localhost:6595`. The links are generated server-side by AudioManager's GUI but resolved by the **viewer's** browser, so opening the GUI from any other device on the LAN yields links pointing at that device's own port 6595. Worth making configurable if David ever browses the GUI from a phone or laptop.
  - A degenerate title made only of punctuation cleans to an empty term, producing `/search?term=` and a blank Deemix search page. Rare enough to leave.
  - Deemix returning zero results is not always a link bug: "Linkin Park Left Behind" legitimately returns nothing because Deezer lacks that track, while "Linkin Park Numb" resolves fine. Worth remembering before re-opening this.

- [x] **Load the queue directly from Liked Songs - capability added (commit `4657ed0`).** `RealSpotifyClient.get_liked_tracks_detailed(limit=None)` reads paginated `/me/tracks` and returns the **identical** row shape to `get_playlist_tracks_detailed` (`{artist, title, album, year, duration_ms}`), so a caller can swap sources with no downstream change. Row building moved into a shared `_detail_row` helper used by both, so the two shapes cannot silently drift. `user-library-read` was already in `SCOPES`, so no re-auth is needed. Verified live read-only against David's real account: one API call, correct shape, and `added_at` confirmed strictly descending, so `limit=N` really is the N most recent likes.

  9 new tests: 8 on the new method (shape parity, multi-artist join, pagination, order preservation, malformed/partial responses, limit capping and stopping pagination early, empty library) and 1 that is the first ever test for `get_playlist_tracks_detailed`, which had none, so the shared-helper refactor is covered on both sides.

  Deliberately **not** done: the method is on `RealSpotifyClient` only, not on the `SpotifyInterface` ABC and not on `SimulatedSpotifyClient`. That exactly matches how the existing `get_playlist_tracks_detailed` is already placed. If AudioManager ever wants this path to work under `--simulate`, both detailed methods need adding to the simulator together - one gap, not two.

  AudioManager hook-in point, for whoever picks that up (nothing in AudioManager was touched): `gui/tabs/acquire.py` already imports from `spotify_tools` and builds its table rows as `(artist, title, album, year, length, url)`. The playlist path is the block near line 219 that calls `get_playlist_tracks_detailed` then `_format_duration` and `_build_deemix_url`. A "Load Liked Songs" button needs the same block with `client.get_liked_tracks_detailed()` substituted for `get_playlist_tracks_detailed(playlist_id)`; every other line, including the Deemix link building and the duration formatting, is unchanged because the row shape is identical. Note `_state["playlist_loaded"]` is used to gate the "extra / unmatched in NewMusic" batch header, so that flag needs a sensible value for the Liked Songs source.

---

Superseded by this plan: the "SpotifyTools generalize - DEFERRED indefinitely" note below (its premise - no second Python consumer - is false; AudioManager's Python GUI is a real, actively-developed consumer) and the "MusicLibPlaylistSyncer" rename idea (narrower name, predates the AudioManager coupling, doesn't reflect current scope).

---

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

- ~~SpotifyTools: generalize as reusable Spotify layer - DEFERRED~~ Superseded 2026-09-02: see ACTIVE PLAN at top of this file. AudioManager's Python GUI is now a real, actively-developed second consumer via direct sibling import.

### Housekeeping
- [x] Fix CLAUDE.md em dashes - done 2026-06-03. Updated project structure and added two-tool scope note at same time.

### Polish

- [ ] Optimize logs for Claude token usage, reduce size
- [x] Rename "NONE" to "NOT FOUND" in CLI display - done 2026-06-03
- [ ] Stricter types - Pylance reports 900+ errors, add type hints gradually
- ~~Rename project - consider "MusicLibPlaylistSyncer"~~ Superseded 2026-09-02: see ACTIVE PLAN at top of this file (renaming to SpotifyTools instead).
- [x] Terminal output: each track on its own line (fixed in 186be2b)
- [ ] Log file results section is huge, format nicer for humans
- [x] `--reset-exhausted` CLI flag - done 2026-06-03. Resets all exhausted tracks to unmatched so they get retried next run.
- [ ] Run AudioManager before running this program so Audio Mirror is up to date
- [x] diagnose.py hint on auth failure - done 2026-06-03. Auth error now suggests running `python src/diagnose.py`.
