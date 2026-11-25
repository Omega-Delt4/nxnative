# LEGO® CITY UNDERCOVER (`010085500130A000`)

Update 1.0.2 (`v131072`) over the base game, SDK 1.3.1, Traveller's Tales' engine on NVN. The shaders are BNSH / BAHS programs inside CC4 `.dat` archives (RFPK-compressed members) and in `main`; `dm/gfx/tools/ttdat.py` reads the archives, and `shader_split.py` maps their control sections and code areas. The Mac app was built on shader chain `8870984a02ba` with all **24,063** keys (22,059 pixel, 2,004 vertex; 23,732 from the `.dat` archives, 331 from `main`; the map has 24,064 data rows, one duplicate key). A long Mac run (about 196,500 frames) had **no** `is not in the shader map` line.

On the Mac: the boot, the title, a new game and the open world; the scripted route below reached the open world, and the app was then played on. The app is 10 GB; its Mac shader library is 1.5 GB and its map 3.4 MB.

## What it needs

| Setting | Where | Why |
|---|---|---|
| `shader_groups /MOVIES/=movies` | [`game.tsv`](game.tsv) | the split groups the `MOVIES` folder's shaders separately |
| `zip_store .dat:.mp4:.metallib` | [`game.tsv`](game.tsv) | `.dat` (RFPK) and `.mp4` are already compressed, so the IPA stores them instead of deflating them again; the Mac app ships `romfs.img` and is not a zip |
| 4 GB layout: `mem_reserve 0x100000000`, `mem_alias 0x10000000`, `mem_stack 0x10000000`, `app_pool 0xCD500000` | [`game.tsv`](game.tsv) | the profile's 4 GB block; the game booted with it |
| `sys_resource 0x0` | [`game.tsv`](game.tsv) | the NPDM declares no system resource, so there is no virtual address memory |
| `modules main subsdk0 subsdk1 subsdk2 subsdk4 sdk` | [`game.tsv`](game.tsv) | `subsdk3` (NintendoSDK_libcurl) is not loaded; `main` takes nothing from it that the host does not supply |
| `romfs_marker GAME.DAT` | [`game.tsv`](game.tsv) | the RomFS's marker |

## Files and shaders

- The RomFS holds thousands of small CC4 `.dat` archives (the shader packages) beside the `MOVIES` folder; their members are RFPK-compressed. `ttdat.py` reads the `-11` / `-12` / `-13` layouts (32-/64-bit name hashes) and decompresses RFPK; `shader_split.py` turns each BNSH / BAHS control section and code area into a key.
- The split found the 24,063 keys above with no `[RED]` lines. The host's key (`hle_nvn.c` `nvn_ctl_lens` / `keys_hash`) uses the same `sha256(control + code)` over the same bytes, so every key the game asked for on the Mac run was found. The old app's dump run found 241 unique missing keys, all in this map; the new app's long run had none (`DM_NVN_SHDUMP`).
- The prewarm list [`pso_manifest.tsv`](pso_manifest.tsv) has no prewarm rows (0 entries; only its header and comments); the host still prewarms the shader libraries (the Mac run's log: 9,396 shaders / 4,015 render pipelines in 13.8 s). `c1_full.tsv` / `cb_extent.tsv` are generated per checkout by a full `dmsh_build.py` run and are not committed.

## Load time

The game loads the small archives and decompresses their RFPK members itself. `zip_store .dat` only keeps the IPA's ZIP from deflating those already-compressed files a second time; the Mac app's `romfs.img` is not recompressed, so the key does not change the runtime decompression. The Mac's load times come from the host and Apple Silicon, and from the complete shader map (no missing-shader fallbacks).

## Mac input

`DM_LAUNCHER=0` does not hide the Mac settings window; pass `--no-launcher` to the app executable to start the game directly. The scripted route `DM_KEYS="8:A,11:A,14:A,17:Plus,20:A,24:A,28:A,32:A"` reached the open world; the long run above was then played with a controller.

## Found on this game

- The CC4 `.dat` archives are read by `ttdat.py`; its `-11` / `-12` / `-13` layouts, 32-/64-bit FNV name hashes and RFPK members are covered by its tests (18 pass).
- The split's `shader_map.tsv` has 24,064 data rows but 24,063 unique keys (one duplicate); the app's map and the build's map are the same.
- `zip_store` is an IPA-packing key, not a runtime decompressor; the Mac app uses the RomFS image as it is.

## Open

- Provisional: the profile and its tables are generated, not hand-reviewed; there is no `scripts/regress.py` baseline or `validated.tsv`.
- No `patches.py`: the generic rewriter ran the tested route without game-specific cuts.
- The prewarm manifest is empty; no `DM_PSO_RECORD` run has been made.
- The open world was played, but not through a full completion run.
