# SpotifyTools - Ideas and TODOs

## ACTIVE PLAN: Rename to SpotifyTools + fix import fragility (2026-09-02)

Scope has grown beyond "playlist gen": this repo is now a Spotify API client + AudioMirror-XML sync engine + interactive playlist browser + acquire-flow orchestration consumed directly by AudioManager's Python GUI (`gui/tabs/acquire.py`, in-process `sys.path.insert` sibling import, not IPC - see AudioManager/docs/References/GUI-Architecture.md "Resolved 2026-08-31": deliberately no separate shared-library repo, revisit only if a second real consumer appears).

Do ONE task at a time, verify before moving to the next. Do NOT batch.

- [x] **Task 1 - package rename (do first, lowest risk):** rename `src/` -> `spotify_tools/` in this repo. Update AudioManager's ~7 `from src.X import Y` lines (`gui/tabs/acquire.py`, `gui/config.py`, `gui/tests/test_acquire.py`) to `from spotify_tools.X import Y`. Run AudioManager's `test_acquire.py` to verify green. Commit both repos separately. Done 2026-09-02: git mv preserved history; SpotifyPlaylistGen commits f714476 (rename) + 98f48df (import/doc updates, 214 tests pass); AudioManager only had import sites in `gui/tabs/acquire.py` (8, not gui/config.py or test_acquire.py) - commit dd3f4a73, 17 tests pass.
- [ ] **Task 2 - path-resolution helper:** replace the hardcoded `SPOTIFYGEN_ROOT = REPO_ROOT.parent / "SpotifyPlaylistGen"` in AudioManager's `gui/config.py` and the hardcoded absolute `config.json` path in `src/config.py`'s `CONFIG_PATH` with one helper that checks a `SPOTIFY_TOOLS_ROOT` env var first, falls back to the sibling-folder assumption. Single point of failure instead of duplicated hardcoding. No packaging/pyproject.toml - not needed, no external consumer.
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
