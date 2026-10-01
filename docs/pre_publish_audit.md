# Pre-publish git-history audit (read-only, no history rewritten)

Date: 2026-10-02. Scope: all 74 HEAD commits (oldest → newest, incl. `0615656`), plus stash entries and current tracked-tree scan.
Method: per-commit `git diff-tree --name-status`, then FULL per-file `git show <hash> -- <path>` (one call per file, no truncated combined show), plus full `git show <hash>:<path>` for new/changed files and binaries. Initial commit (`cef5fe6`) read in full via `git ls-tree -r` + per-file `git show`. Five parallel chunk audits (1–15, 16–30, 31–45, 46–60, 61–73/HEAD). No files or history modified for this audit. Working-tree `git diff` left untouched.

Goals in priority order: (1) no copyrighted game data ever committed, (2) no third-party files that should have been gitignored, (3) other cleanup.

## Executive summary

1. **Copyrighted game data: NOT CLEAN — 1 live blocker.** `src/extras.c` in the current working tree still embeds two verbatim ROM-derived byte arrays (verified 2026-10-02: 512 + 64 entries):
   - `title_screen_tiles[512]` — introduced in `3e89ad9` (#17), edited in `20c9b28` (#45), still present at HEAD (line 52).
   - `attr_table[64]` — introduced in `f37bb8a` (#26, comment cites "ROM bank 0 offset 0x073C (64 bytes)" → VRAM `$23C0–$23FF`), still present at HEAD (line 176).
   Both are bulk creative ROM content (tile/nametable/attribute bytes; the 512 tail also reads as raw 6502 code bytes), not small facts. Everything else reviewed (addresses, single register bytes, CRC32/SHA-1, mapper/bank counts, iNES header magic, frame hashes, disassembly *mentions*, generic 6502 opcode tables, NES palette facts, button-mask constants) is correctly classified as non-copyrightable facts/tools and is CLEAN.
2. **Third-party / gitignore: CLEAN with intentional exceptions to confirm.** No ROMs (`.nes`), screenshots, dumps, `venv/`, `build/`, `generated/`, `external/mesence` outputs, or blargg test ROMs were ever tracked. `nesrecomp`/`mesence`/`mesen-for-ai` existed only as submodule pointers (never vendored). `tools/nes_compare/` was never tracked. blargg ROMs are fetch-only (`external/nes-test-roms/` gitignored, untracked). The only tracked non-source blobs are two intentional TAS input logs at HEAD: `tas/vectors/sdb_4976.bk2` (TASVideos #4976 input timings, no ROM bytes) and `tas/vectors/sample_smb2_warp.fm2` (SMB2-J parser fixture, input text + emulator-state blob, no ROM bytes) — attribution already in `tools/tas/README.md`; confirm intent to ship them (they match no gitignore rule, so tracked by design).
3. **Other cleanup:** no secrets found. Several non-blocking hygiene items listed below (docs duplication, stale commit messages, exec-bit churn, blanket `external/mesence/` ignore, local-only stash entries, uncommitted working-tree edits).

## Current-tree risky-file scan (2026-10-02)

- `git ls-files | grep -Ei '\.(nes|fds|gb|gbc|smc|sfc|gen|zip|png|ppm|bin|pal|tga|srm|sav|bk2|fm2)$'` → only the two TAS vectors above. No `.nes`, screenshots, `.bin`/`.ppm` dumps, ROMs.
- Prefix scan (`build|venv|generated|external/mesence|external/nes-test-roms|config|logs|tmp|recomp_output|nes_reference|tas/runs`) → only `build_and_run.sh` (false positive: source script starting with "build"). All artifact dirs properly untracked.
- `git stash list` → two local-only entries on top of `a416e5d` (`WIP on master`, `index on master`); not in HEAD history, never pushed by default. They touch only `src/extras.c` variants (same 512-byte array + trivial `g_ppumask_translate` flip). No ROMs/dumps. Optional hygiene: drop before publishing.
- Working tree at audit time had 3 uncommitted modifications (`build_and_run.sh`, `tools/tas/README.md`, `tools/tas/run_tas.py`, 71 insertions); these were committed as `0615656` before the remediation sizing was written. Current working tree holds only this audit file (`?? docs/pre_publish_audit.md`).

## Full commit verdicts (74 HEAD commits)

Ratings: CLEAN / CARRYOVER (pre-existing risk still present, diff adds none) / COPYRIGHT-RISK (introduces or edits verbatim ROM bytes) / NOTE (third-party or cleanup flag, still clean for copyright).

| # | Hash | Subject | Verdict |
|---|------|---------|---------|
| 1 | cef5fe6 | Initial project structure | CLEAN (hashes + 16B header + specs = facts) |
| 2 | 658394f | NESRecomp framework, boots | CLEAN (gitlink pointer; note: hardcoded `/usr/lib/…libSDL2` path) |
| 3 | e5cb211 | game_run_main → func_RESET | CLEAN |
| 4 | b415a5b | PPUMASK override + OAM fix | CLEAN (OAM memcpy = live runtime state, not ROM) |
| 5 | 9ae4d94 | PPUMASK translation, optional OAM | CLEAN |
| 6 | 10ca48f | AGENTS: translation layer | CLEAN |
| 7 | ebe31a9 | AGENTS: correct frame hash | CLEAN |
| 8 | 0ce225d | g_ppumask=0x1E | CLEAN |
| 9 | c3318e8 | AGENTS: translation details | CLEAN |
| 10 | d4da863 | submodule: drop debug prints | CLEAN (pointer) |
| 11 | d2c32fd | submodule: drop runtime prints | CLEAN (pointer) |
| 12 | 376348d | AGENTS: current state | CLEAN |
| 13 | faddbd7 | PPUMASK always-enabled | CLEAN |
| 14 | 8dee015 | AGENTS compact rewrite | CLEAN |
| 15 | ab934af | bypass NMI dispatcher | CLEAN (function *calls* by name, no bytes) |
| 16 | 411a093 | AGENTS: title rendering | CLEAN |
| 17 | 3e89ad9 | Load 512-byte tile data | **COPYRIGHT-RISK (new: `title_screen_tiles[512]`, bank-1 `$031F–$0585` per comments)** |
| 18 | 01923ad | memcpy tile init | COPYRIGHT-RISK (carryover; msg wrongly says AGENTS.md) |
| 19 | aa21c82 | AGENTS research findings | CLEAN |
| 20 | 3c9ad00 | PPUCTRL=0 + memcpy | COPYRIGHT-RISK (carryover) |
| 21 | daf3672 | AGENTS PPUCTRL/empty-funcs | CLEAN (3-byte `04 04 04` illustrations = facts) |
| 22 | 02708c4 | Stable baseline | CARRYOVER (extras) / CLEAN (toml address list) |
| 23 | 73f07b0 | AGENTS stable baseline | CLEAN |
| 24 | 95e1881 | Force PPUCTRL 0x10 | CARRYOVER |
| 25 | 71c38c6 | AGENTS major fixes | CLEAN (notes untracked generated-code patching hygiene) |
| 26 | f37bb8a | Attribute table from ROM 0x073C | **COPYRIGHT-RISK (new: `attr_table[64]`; file now 512+64)** |
| 27 | c50124b | AGENTS attr-table note | CLEAN (duplicated "Applied Patches" heading — cleanup) |
| 28 | a04d4cd | venv/ gitignore | CLEAN (hygiene) |
| 29 | 475630e | build_and_run.sh | CLEAN |
| 30 | 37617bf | AGENTS round-trip results | CLEAN |
| 31 | fd4cafd | round-trip pipeline + tooling | CLEAN (`rom_to_asm.py`/`run_roundtrip.py` read ROM at runtime, zero embedded bytes) |
| 32 | a1a7ef2 | AGENTS cleanup | CLEAN |
| 33 | 4bc81e2 | PPUMASK patch + setup.sh | CLEAN (patch is engine-only) |
| 34 | f2e25ac | remove logs/docs/gitignore | CLEAN (title claims log removal; diff has zero deletions) |
| 35 | d1ab619 | game.toml + ppuctrl backup | CLEAN |
| 36 | 4af6621 | CMake recompiler step | CLEAN |
| 37 | 39fd0e4 | tile-loading flag | CLEAN |
| 38 | 251908e | commit rule/gitignore | CLEAN (msg claims setup.sh change not in diff) |
| 39 | 94d7d66 | logs/ gitignore | CLEAN |
| 40 | 37a4c78 | AGENTS/gitignore/setup | CLEAN (`tools/nes_compare/` referenced but never tracked — no vendoring) |
| 41 | d3cf2c5 | --save-screenshot flag | CLEAN (palette = generic NES hardware facts) |
| 42 | dee599b | plastic_core→MesenCE | CLEAN (submodule pointers only) |
| 43 | de439bb | remove mesen-for-ai | CLEAN (note: blanket `external/mesence/` ignore vs tracked gitlink — harmless but sloppy) |
| 44 | db66caf | config/ + Lua API fix | CLEAN (zero `config/`/`.nes`/`.png` tracked) |
| 45 | 20c9b28 | title_screen_tiles fix | **COPYRIGHT-RISK (edits 512-B array + 100+ B 6502 code-byte tail)** |
| 46 | 51b619c | Overhaul docs, delete plan.md | CLEAN |
| 47 | 3194d28 | g_ppumask toggle | CARRYOVER (diff is 8 lines; arrays persist in-file) |
| 48 | ea6e218 | conditional ppumask translate | CLEAN |
| 49 | 73bbfbe | Remove 001 patch (upstreamed) | CLEAN |
| 50 | d34e532 | AGENTS: remove patch refs | CLEAN |
| 51 | eed4953 | Remove patching process | CLEAN |
| 52 | 2c490ec | todo list rule | CLEAN |
| 53 | 903dcef | efficiency rules | CLEAN |
| 54 | 37f7f3f | accurate extras hooks | CLEAN |
| 55 | 293b94c | Consolidate documentation | CLEAN (1-line Zelda `ASL A; TAY` annotation, no game excerpts) |
| 56 | 1fc00c8 | verification + disasm tooling | CLEAN (capstone/py65 = PyPI deps, not vendored) |
| 57 | e423f74 | iterative testing procedure | CLEAN (note: `git checkout -- src/…` without backup — docs safety issue) |
| 58 | 8399642 | Phase 2C 3-frame hang | CLEAN |
| 59 | ccaab01 | func_FF08 replace_func | CARRYOVER |
| 60 | cd1f548 | extra_func + force_interp | CARRYOVER (toml = addresses only; plan.md paste duplication — cleanup) |
| 61 | 90e1c6a | Update plan.md | CLEAN |
| 62 | 088c2ac | dispatch_override analysis | CLEAN |
| 63 | 216b217 | frame/hang investigation | CLEAN |
| 64 | a416e5d | nesrecomp → origin/master pr#58 | CLEAN (pointer `cfc483f→3ef948c`) |
| 65 | 27b8710 | NMI/func_NMI notes | CLEAN (`LDX #$FF; TXS` = functional facts) |
| 66 | 92389e1 | Title done 99.8% + dis65 + oracle_dump | CARRYOVER (new files clean; `oracle_dump.lua` *writes* dumps at runtime, commits none) |
| 67 | bcfac0f | README rewrite | CLEAN (hashes/specs = facts) |
| 68 | a0c41ba | Archive NMI-tail patch | CLEAN (pure rename + revival-conditions doc) |
| 69 | fe7b98a | input-frontier dossier §18 | CLEAN (register snapshots/addresses = facts) |
| 70 | bbf0773 | Input past title, patches 002–004 | CLEAN (patches = original runner C; input script = original) |
| 71 | b3a94c0 | Split interp patch 002/003 | CLEAN |
| 72 | 96b9f81 | blargg engine suite | CLEAN w/ NOTE (test ROMs fetched via opt-in `--fetch`, never vendored; keep "fetched, not shipped" prominent) |
| 73 | 349f72c | TAS harness + vectors (HEAD) | CLEAN w/ NOTE (vectors are third-party *input logs*, no ROM bytes; see below) |
| 74 | 0615656 | Rebuild when inputs are stale (TAS runs, build_and_run.sh) | CLEAN (original build-wiring code + docs; no game data, no vendoring — per-file diffs read in full) |

## Detail: copyright blocker timeline

- `3e89ad9` adds `static const uint8_t title_screen_tiles[512]` (16×32-byte blocks, comments claim bank 1 `$031F–$0585`), copied via `nes_write(0x0108+i,…)`. Block labels 11–15 are incoherent/overlapping and blocks 9–12/15 read as 6502 code bytes (`A5 85 28 60 4C…`) — misattributed but still verbatim ROM bytes.
- `01923ad` → `95e1881` keep the array byte-identical (copy-mechanism/logic changes only).
- `f37bb8a` adds `static const uint8_t attr_table[64]` (`0x03×5, 0x02×7, 0x01×12, 0x00×40`), written to PPU `$23C0–$23FF`.
- `20c9b28` edits array blocks 6–7/12–15 and leaves a ~100+ byte raw-opcode tail (`A5 71 0A A8 B9 B9 85 85 … 4C B2 85`).
- `3194d28` → HEAD: diffs are logic-only but the file retains both arrays verbatim (confirmed 512 + 64 entries in working tree).
- De minimis context: ~576 bytes total (~0.2% of PRG), but it is literal game content in tracked source — the decision for publication is either runtime extraction (read from user-supplied ROM, consistent with how `rom_to_asm.py`/`oracle_dump.lua` already behave) or a documented fair-use/ownership basis. No history rewrite performed per instructions.

Explicitly NOT flagged (facts/tools): CPU addresses (`$FF08`, `$FE8A`, `$FCA0`, `$8393`, `$031F–$0585`, `$073C`, `$0106/$0107`, `$2000/$2001/$2006/$2007`, `$4016`), single-byte values (`0x1E`, `0x10`, `04 04 04` 3-byte illustrations), CRC32/SHA-1/MD5, mapper/bank/mirroring specs, iNES magic `4E 45 53 1A`, frame hashes, `func_*` names, generic opcode tables (`rom_to_asm.py` 256-entry table, `tools/dis65.py`, capstone example), stock NES palette, TAS button masks, RAM-address watchlists.

## Detail: third-party / gitignore timeline

- Never tracked at any checked commit: `.nes`, screenshots (`.png`/`.ppm`), `.bin`/`.pal`/`.tga` dumps, `venv/`, `build*/`, `generated/`, `external/mesence/` outputs, `external/nes-test-roms/`, `config/`, `logs/`, `tmp/`, `recomp_output/`, `nes_reference/`, `tas/runs/`, `tas/*.tas.json`, `tools/blargg/toml/`, `canary.nes`. `.gitignore` covered ROM/build/generated/log patterns from commit 1 and was tightened correctly at `658394f`, `e5cb211`, `fd4cafd`, `f2e25ac`, `251908e`, `94d7d66`, `db66caf`, `bbf0773`, `96b9f81`, `349f72c`.
- Submodules are pointers only (no vendored code): `external/nesrecomp` (multiple pointer bumps, incl. PR #22 PPUMASK fix and `a416e5d` pr#58), `external/mesence`, `external/mesen-for-ai` (added `dee599b`, removed `de439bb`).
- `tools/nes_compare/` (Rust) referenced by `setup.sh`/`AGENTS.md` around `37a4c78` but `git log --all -- tools/nes_compare/` is empty — never tracked.
- `pyproject.toml` deps (`nesasm`, `capstone`, `py65`) are package-manager declarations, not vendored code.
- `patches/*.patch` (001 PPUMASK, 002–005 interp/NMI/watchdog) are original runner C — clean.
- `tools/frame_gen/*.lua`, `tools/blargg/*`, `tools/tas/*` are original harness/probe code — clean. `oracle_dump.lua`/`make_canary.py` generate dumps/ROMs at runtime; outputs are gitignored and untracked.
- TAS vectors (HEAD): `sdb_4976.bk2` (10,614 B zip: `Header.txt` metadata + ~19.7k input lines, headerless-ROM MD5 `6e44871b…` as fact; no ROM/CHR/savestate blob beyond stock BizHawk JSON) and `sample_smb2_warp.fm2` (1032-line FCEUX input text + emulator-state blob; documented parser fixture, not harness-runnable). Negligible copyright risk (functional input timings); attribution present in `tools/tas/README.md`. Confirm intent to publish; optionally add `tas/vectors/README` pointer and/or gitignore-rule clarification since `.bk2`/`.fm2` match no ignore rule (tracked by design, not by gap).

## Other cleanup (non-blocking, no fixes applied)

- `src/CMakeLists.txt` (#2–#15 era): hardcoded `/usr/lib/x86_64-linux-gnu/libSDL2-2.0.so.0` — verify replaced with `find_package`/pkg-config at HEAD before publishing.
- `game.toml` `fixed` lists contain opcode-looking 2-byte guesses (`0xA901`/`0x8500`-class) — later project guidance already bans these (function-split risk); correctness note only.
- Commit-message inaccuracies: `01923ad` (says AGENTS.md, touches only `extras.c`), `251908e` (says setup.sh, no setup.sh in diff), `f2e25ac` (says removes logs, zero deletions).
- `build_and_run.sh` exec-bit churn (#34 off, #36 on).
- Duplicated "Applied Patches" heading after `c50124b`; pasted-duplicate checklist in `docs/plan.md` after `cd1f548`; `e423f74` testing procedure recommending bare `git checkout -- src/…` (later revised to back up to `/tmp` first).
- Blanket `external/mesence/` ignore vs tracked gitlink (#43) — harmless but sloppy.
- Tile-array block-label incoherence (#17: labels 11–15 overlapping `$0587`/`$0585`/`$0185`/`$014C`/`$C9BC`) — re-verify provenance before any keep/remove decision.
- Local-only `stash@{0,1}` and 3 uncommitted working-tree files — review/drop or commit deliberately; not part of published history until committed.

## Suggested pre-publish actions (not done — documentation only)

1. Resolve `title_screen_tiles[512]` + `attr_table[64]` in `src/extras.c` (runtime extraction from user ROM, or documented basis). This is the only copyright blocker found.
2. Confirm intent to ship `tas/vectors/*.bk2|*.fm2`; optionally add `tas/vectors/README` with provenance/license note (already attributed in `tools/tas/README.md`).
3. Verify HEAD `src/CMakeLists.txt` has no machine-specific SDL2 path; verify no secrets in full-tree grep.
4. Decide stash disposition (drop) and working-tree disposition (commit or revert) before `git push`.
5. Then choose history strategy (redaction of `3e89ad9`/`f37bb8a`/`20c9b28` lineage vs. fresh squash) — explicitly out of scope for this audit.

## Local remediation sizing vs HEAD (added 2026-10-02; no code changed)

Each audit blocker/cleanup item compared against the current working tree (`0615656`), with removal distance and blast radius. Verified via `grep`/reads of `src/extras.c`, `src/game.toml`, `src/CMakeLists.txt`, `tools/tas/README.md`, `.gitignore`, `AGENTS.md`.

### 1. `title_screen_tiles[512]` — DEAD CODE, trivial working-tree fix, history still tainted
- Local state: declared `src/extras.c:52–117`, **zero uses** (`grep` hits only the declaration; the old `memcpy(&g_ram[0x0108], title_screen_tiles, 512)` is gone — `game_post_nmi` comment lines 199–202 explicitly documents its removal as a stack-smashing bug, real NMI path via `$F700`/`$E49C` stages `$0108` instead). Title renders ~99.8% without it.
- Working-tree fix distance: delete the 67-line static array, nothing else references it. Rebuild + `smoke 5` + frame-11 compare is the full regression gate; expected zero behavioral delta.
- Ramification caveat: working-tree deletion does NOT clean the published history — `3e89ad9`/`01923ad`/`20c9b28` blobs remain reachable in git objects. Publishing still needs history redaction (e.g. `git filter-repo` on that lineage, or fresh squash) even after the local delete. The embedded 6502-code-byte tail (`A5 71 0A A8…`) confirms ROM-dump provenance, so don't reclassify as coincidental.

### 2. `attr_table[64]` — LIVE BOOT-PATH CODE, small but load-bearing fix
- Local state: `src/extras.c:176–188`, written once in `game_on_init()` to PPU `$23C0–$23FF` (comment: "Without this, attribute table is all zeros → 2-color display"). Unlike the tile array, this one executes every boot.
- Fix option A (recommended for publish safety): runtime extraction — read 64 B from the user-supplied ROM at init (comment claims PRG bank 0 offset `0x073C` → file offset `16 + 0x073C` for a 16-byte iNES header; parse the header rather than hardcoding). Precedent exists in-tree (`rom_to_asm.py`, `oracle_dump.lua` already read the ROM at runtime and commit no bytes). Distance: ~20 lines in `game_on_init()` plus error handling.
- Fix option A ramifications: `g_rom_path_for_extras` is currently `""` (`extras.c:39`) — verify how/whether the runner populates it before `game_on_init`, or plumb the ROM path explicitly; a failed/mis-parsed read breaks palette output on *every* boot (silent 2-color regression), so the gate must be `smoke 5` + frame-11 color compare, ideally plus the menu path (mode/skill/team-select, 330+ frames) since attributes affect all nametables. There is also an unresolved model tension: `AGENTS.md` says `func_FF61` zeroes `g_ppu_nt` every frame, which would not survive an init-once write — yet the title is stable at 99.8%, so either the real NMI path restores attributes post-boot or that claim is stale. Test both: (i) delete-and-replace, (ii) if colors survive a no-write boot, the table may be redundant and the fix collapses to deletion like item 1.
- Fix option B: keep + document a de minimis/functional basis (64 B palette-routing bytes, ~0.05% of PRG). Zero code risk, but the publish risk stays.

### 3. `src/CMakeLists.txt:22` SDL2 absolute path — STILL PRESENT, low-risk portability fix
- Local state: `set(SDL2_LIBRARY /usr/lib/x86_64-linux-gnu/libSDL2-2.0.so.0)` unchanged at HEAD. Builds on this machine, breaks fresh clones on other distros/paths.
- Fix distance: replace with `find_package(SDL2)` / pkg-config or the already-vendored `src/third_party/SDL2` + runner-bundled SDL2 include wiring (both already referenced lines 17–19/50–52). Ramification: rebuild-and-run locally to confirm linkage; no game-logic effect.

### 4. `game.toml` opcode-guess entries — ALREADY RESOLVED, no action
- Local state: `fixed` is 6 entries (`FF61 FE96 FFBE FE34 E49C E4E5`), `extra_func` are dis65-verified bank/address pairs, `[force_interp] fixed = []`. The historical `0xA901`/`0x8500`-class concern does not apply to HEAD (`grep` empty).

### 5. TAS vectors — KEEP, optionally add pointer file (trivial, no code effect)
- Local state: `tas/vectors/sdb_4976.bk2` (10 KB) + `sample_smb2_warp.fm2` (71 KB) tracked by design; attribution present (`tools/tas/README.md` names ShesChardcore & Dasrik, `tasvideos.org/4976M`, ROM MD5-as-fact, SMB2 parser-fixture-only note). No ROM/CHR/savestate bytes.
- Removal ramifications: deleting `.bk2` breaks canonical-JSON regeneration (`bk2_to_tas.py` → gitignored `tas/*.tas.json`); deleting `.fm2` breaks `fm2_to_tas.py` parser tests. Keep both; optional `tas/vectors/README` pointer is docs-only.

### 6. Remaining cleanup items — status at HEAD
- Blanket `external/mesence/` ignore (`.gitignore:75`) vs tracked gitlink: still present, harmless (affects only untracked build outputs inside the submodule workdir). Optional one-line tighten, zero risk.
- `stash@{0,1}` (pre-`a416e5d` `extras.c` variants): still local-only, still unpushed; drop before publishing (one command, no code effect).
- Prior working-tree edits (`build_and_run.sh`, `tools/tas/README.md`, `tools/tas/run_tas.py`): absorbed by `0615656` — working tree is clean except this audit file (`?? docs/pre_publish_audit.md`).
- Historical notes verified resolved: `build_and_run.sh` is `+x` (exec-bit churn over), single "Applied Patches" heading (`AGENTS.md:174`), backup-before-checkout rule present (`AGENTS.md:94-95,200`), `tools/nes_compare/` still never tracked.
- New commit `0615656` itself: `ensure_fresh_build()` + staleness regen + `--no-build` — original code, no game/third-party content. Its only audit relevance is process: TAS runs now rebuild on `game.toml`/`extras.c`/`CMakeLists.txt` staleness, so remediation edits above will actually take effect in harness runs instead of silently testing a stale binary.

## Mentioning vs embedding game data + industry practice (added 2026-10-02)

Short version: *mentioning* tile data (addresses, sizes, hashes, "title uses 512 B from bank 1") is not the risk — every reference project does exactly that. *Embedding* the 512 + 64 verbatim bytes in `src/extras.c` is the risk. (Not legal advice; consult counsel for a publish decision.)

- **Mentioning = facts, universally done.** Our docs/`game.toml`/README references — CPU addresses (`$031F–$0585`, `$073C`), byte counts, CRC32/SHA-1/MD5, mapper/bank specs, frame hashes — are non-copyrightable facts. Reference projects publish the same class of data openly: SuperMarioBrosNESRecomp's README lists only the ROM's CRC32/MD5/SHA-1 plus "select your ROM when prompted"; Ship of Harkinian / Shipwright docs say a legally acquired ROM is required and the launcher verifies it; SM64-family decomps state the repo holds only original work and no ROM/assets.
- **Embedding = copying expression, universally avoided.** The industry line is consistent — ship hashes + extraction code, never the bytes:
  - Ship of Harkinian: ships no copyrighted assets; the launcher generates the `.otr` asset file from the user's ROM at install time.
  - SuperMarioBrosNESRecomp (same NESRecomp ecosystem as this project): generated C is committed, but the game still needs the user's ROM at runtime; every owner-ROM mod page repeats the formula — "extracted into memory from that verified ROM. No [game] ROM data or derived graphics/audio are shipped or written beside the game" (Samus/Zelda II/Sonic/Smash 64 mods).
  - SM64 decomp family: "contains no ROM and no extracted Nintendo assets"; assets are extracted from a prior copy of the game at build time.
- **Where our arrays fall.** The 512-B `title_screen_tiles` + 64-B `attr_table` are verbatim ROM content used as a functional substitute for ROM bytes (the recompiler/runner equivalent of shipping a slice of the `.otr`). That it is ~0.2% of PRG does not make it safe: de minimis is a narrow doctrine, not a byte-count threshold, and the fourth fair-use factor (substitution for the original, even partially) cuts against excerpts consumed as live game data rather than quoted for comment/criticism. It might ultimately be defensible, but it is the exact category of content that gets DMCA-noticed on GitHub, and Nintendo is the most aggressive filer in this space — so the audit treats it as a blocker while treating all mentions/facts as clean.
- **Practical consequence for this repo:** keep the mentions (they are the documentation), fix the embeds per the sizing section above — delete the dead 512-B array outright, convert the live 64-B table to runtime ROM extraction like `rom_to_asm.py`/`oracle_dump.lua` already do. That puts the project exactly on the industry pattern: config + code + hashes in git, bytes from the user's ROM at build/run time.

## Remediation playbook — step-by-step without breaking the game (added 2026-10-02; NOT YET EXECUTED)

Prerequisites verified: `g_rom_path_for_extras = argv[1]` is assigned in `main_runner.c:2149-2150` *before* `game_on_init()` (`:2157`), so init-time ROM reads are viable. Gate every step on the AGENTS.md procedure: back up to `/tmp/` first (never bare `checkout`/`restore`/`stash`), force regen after `game.toml` edits (`NESRecomp … --game src/game.toml`, confirm `[Eval] Coverage`), rebuild, `timeout 25 ./build/super_dodgeball "roms/Super Dodge Ball (USA).nes" --smoke 5 --smoke-interval 1` must emit JSON with `frames_run ≥ 5` and `dispatch_miss_count == 0`, then frame-11 vs Mesen `nes_reference/` compare (target ≥90%, current ~99.8%). Do steps in order, one at a time, commit only when asked.

### Fix 1 — delete `title_screen_tiles[512]` (dead code, zero behavior change)
1. `cp src/extras.c /tmp/extras.c.fix1.bak`
2. Delete `src/extras.c:51-117` (the `/* Forward declaration */` comment + full `static const uint8_t title_screen_tiles[]` array). Touch nothing else — grep must show zero remaining `title_screen_tiles` hits.
3. `cmake --build build -j$(nproc)` (no regen needed; `game.toml` untouched).
4. Gate: `smoke 5` JSON green + frame-11 compare. Expected delta: none (array has no readers; title comes from the real NMI path `$F700`/`$E49C`).
5. If red (shouldn't be): `cp /tmp/extras.c.fix1.bak src/extras.c`, rebuild, re-run — back to known-green.

### Fix 2a — probe the attr source (read-only, no game change)
1. With system `python3`, read 64 B at file offset `16 + 0x073C` from `roms/Super Dodge Ball (USA).nes` (16-byte iNES header, no trainer on this ROM — verify header[6] bit 2 is clear) and byte-compare against the `attr_table` literal (`extras.c:176-185`). Also confirm via `python3 tools/dis65.py` that the surrounding region is data, not code.
2. Proceed only on exact match. If it mismatches, stop — provenance assumption is wrong and the extraction offset must be re-derived, not guessed.

### Fix 2b — runtime-extract `attr_table[64]` in `game_on_init()` (the load-bearing one)
1. `cp src/extras.c /tmp/extras.c.fix2.bak` (separate backup; Fix 1 must already be green).
2. Add a small static helper in `extras.c`: open `g_rom_path_for_extras`, parse the 16-byte iNES header (skip 512-B trainer iff header[6] bit 2 set), `fseek` to `prg_base + 0x073C`, `fread` 64 B into a local `uint8_t buf[64]`. On any failure: `fprintf(stderr, …)` loud + `memset(buf, 0)` so the failure mode is the already-known 2-color title (frame compare catches it) rather than a silent wrong-palette boot. No embedded fallback bytes — that would defeat the fix.
3. Replace the `static const uint8_t attr_table[64] = {…}` block with the helper call; keep the `nes_write(0x2006, 0x23/0xC0)` + `for (i<64) nes_write(0x2007, buf[i])` loop byte-identical.
4. Rebuild + `smoke 5` JSON gate + frame-11 color compare (this is the palette fix — watch the structural score *and* eyeball color vs Mesen 11), then the menu regression (`tools/input/title_to_team_select.txt` path to team-select, 330+ frames, 0 misses) since attributes affect every nametable, not just the title.
5. Open question the test answers: if a no-write boot *also* stays colorful, the init-once write is redundant against the real NMI path (tension with the `func_FF61`-zeroes-nametable claim in AGENTS.md) and the fix collapses to deletion. Either outcome is publish-clean.
6. If red: restore `/tmp/extras.c.fix2.bak`, rebuild, re-run.

### Fix 3 — `CMakeLists.txt:22` SDL2 path (portability, after game fixes are green)
1. Replace `set(SDL2_LIBRARY /usr/lib/…)` with `find_package`/pkg-config or the already-referenced bundled `src/third_party/SDL2` + runner SDL2 includes (lines 17-19/50-52 already wire the headers).
2. Full clean rebuild on this machine + `smoke 5`. No game-logic effect; failure mode is link-time, not runtime.

### Fix 4 — publish hygiene (no game effect)
- Keep `tas/vectors/*.bk2|*.fm2` (removal breaks `bk2_to_tas.py` regen + `fm2_to_tas.py` parser tests); optional docs-only `tas/vectors/README` pointer.
- `git stash drop` the two local-only entries (verify `git stash list` scope first).
- History redaction (`3e89ad9`/`f37bb8a`/`20c9b28` lineage via `filter-repo`, or fresh squash) happens *after* Fixes 1-2 are green and committed — redaction is a publish step, never a debugging step.
