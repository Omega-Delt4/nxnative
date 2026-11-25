# The LEGO® NINJAGO® Movie Video Game (`01000CE002072000`)

Base 1.0.0 (`v0`), SDK 1.3.1, Traveller's Tales' engine on NVN. The update 1.0.3 (`v196608`) has the same 7,257 shader keys (the map files are identical), but this profile is for the base v0 and the update was not swept. The shaders are BNSH / BAHS programs inside CC4 `.dat` archives (RFPK-compressed members) and in `main`; `dm/gfx/tools/ttdat.py` reads the archives, and `shader_split.py` maps their control sections and code areas. The Mac app was built on shader chain `8870984a02ba` with all **7,257** keys (5,406 pixel, 1,851 vertex; 6,826 from the `.dat` archives, 431 from `main`; the map has 7,258 data rows, one duplicate key). A 90 s Mac run (2,605 frames, 211 pipeline bases) had **no** `is not in the shader map` line; it did not reach a full level.

On the Mac: the boot, the title and the opening; the app is 7.8 GB, its Mac shader library 498 MB and its map 956 KB.

## What it needs

| Setting | Where | Why |
|---|---|---|
| `shader_groups /=romfs` | [`game.tsv`](game.tsv) | all shader packages are grouped as the RomFS package |
| `zip_store .dat:.metallib` | [`game.tsv`](game.tsv) | `.dat` (RFPK) is already compressed, so the IPA stores it instead of deflating it again; the Mac app ships `romfs.img` and is not a zip |
| 4 GB layout: `mem_reserve 0x100000000`, `mem_alias 0x10000000`, `mem_stack 0x10000000`, `app_pool 0xCD500000` | [`game.tsv`](game.tsv) | the profile's 4 GB block; the game booted with it |
| `sys_resource 0x0` | [`game.tsv`](game.tsv) | the NPDM declares no system resource, so there is no virtual address memory |
| `modules main subsdk0 subsdk1 subsdk2 subsdk3 subsdk4 sdk` | [`game.tsv`](game.tsv) | `subsdk5` (C / libc) is not loaded; `main` takes nothing from it that the host does not supply |
| `mem_used 0x6a00000` | [`game.tsv`](game.tsv) | the rewritten modules are larger than the default 64 MB of used memory, and the game's `SetHeapSize` asks for pool − used |
| `romfs_marker GAME.DAT` | [`game.tsv`](game.tsv) | the RomFS's marker |

## Files and shaders

- The RomFS holds thousands of small CC4 `.dat` archives (the shader packages); their members are RFPK-compressed. `ttdat.py` reads the `-11` / `-12` / `-13` layouts (32-/64-bit name hashes) and decompresses RFPK; `shader_split.py` turns each BNSH / BAHS control section and code area into a key.
- The split found the 7,257 keys above with no `[RED]` lines. The host's key (`hle_nvn.c` `nvn_ctl_lens` / `keys_hash`) uses the same `sha256(control + code)` over the same bytes, so every key the game asked for on the Mac run was found.
- The prewarm list [`pso_manifest.tsv`](pso_manifest.tsv) has no prewarm rows (0 entries; only its header and comments); the host still prewarms the shader libraries (the Mac run's log: 1,767 shaders / 1 render pipeline in 108.6 ms). `c1_full.tsv` / `cb_extent.tsv` are generated per checkout by a full `dmsh_build.py` run and are not committed.

## Load time

The game loads the small archives and decompresses their RFPK members itself. `zip_store .dat` only keeps the IPA's ZIP from deflating those already-compressed files a second time; the Mac app's `romfs.img` is not recompressed, so the key does not change the runtime decompression. The Mac's load times come from the host and Apple Silicon, and from the complete shader map (no missing-shader fallbacks).

## Mac input

`DM_LAUNCHER=0` does not hide the Mac settings window; pass `--no-launcher` to the app executable to start the game directly. The scripted route `DM_KEYS="8:A,11:A,14:A,17:Plus,20:A,24:A,28:A,32:A,40:A,50:A"` was used for the 90 s run above.

## Found on this game

- The CC4 `.dat` archives are read by `ttdat.py`; its `-11` / `-12` / `-13` layouts, 32-/64-bit FNV name hashes and RFPK members are covered by its tests (18 pass).
- The split's `shader_map.tsv` has 7,258 data rows but 7,257 unique keys (one duplicate); the app's map and the build's map are the same.
- The update v196608's map is byte-identical to the base v0 map (7,257 keys), so the shader work carries over; the update's `main` build and `mem_used` were not run.
- `zip_store` is an IPA-packing key, not a runtime decompressor; the Mac app uses the RomFS image as it is.

## Open

- Provisional: the profile and its tables are generated, not hand-reviewed; there is no `scripts/regress.py` baseline or `validated.tsv`.
- No `patches.py`: the generic rewriter ran the tested route without game-specific cuts.
- The prewarm manifest is empty; no `DM_PSO_RECORD` run has been made.
- The 90 s run did not reach a full level; the update v196608 (1.0.3) was not swept.
