# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Kodi skin** (a GUI theme add-on) — a fork/derivative of Kodi's default **Estuary** skin, packaged as `skin.leanback`. There is no application source code, compilation step, or test suite. The entire UI is declarative **Kodi skinning XML**: windows, dialogs, and reusable includes interpreted by Kodi's GUI engine at runtime.

Note the identity mismatch: `addon.xml` still declares `id="skin.estuary"` / `name="Estuary"` while the repo and git remote are `skin.leanback`. Preserve existing IDs unless a change is explicitly requested — changing the addon id breaks upgrades for installed users.

## Working with the skin (build / test / run)

There is no build or lint. To see changes, load the skin into Kodi:

- Copy or symlink this folder into Kodi's `addons/` directory as `skin.estuary` (matching the addon id), then select it in Settings → Interface → Skin.
- After editing XML, reload without restarting Kodi: **Settings → Interface → Skin → hold/right-click the skin → "Reset above settings to default"** is not it — use the debug reload: with `debugging="true"` in `addon.xml`'s `<extension point="xbmc.gui.skin">`, press the reload key, or run the built-in `ReloadSkin()` (map to a key or execute via JSON-RPC). Kodi logs XML parse errors to `kodi.log`.
- `<res>` entries in `addon.xml` all point at the single `folder="xml"`; there is no per-resolution XML — layouts scale via coordinates against the base 1920×1080 (16:9 is `default="true"`).

### Textures (`media/*.xbt`)

`media/Textures.xbt`, `curial.xbt`, and `flat.xbt` are **compiled texture bundles**, not editable files. They are produced from source PNGs with Kodi's `TexturePacker` tool (`TexturePacker -input <dir> -output media/Textures.xbt`). To change an image, unpack/repack with TexturePacker — you cannot edit `.xbt` directly. Loose textures under `extras/`, `themes/`, and `resources/` are used as-is.

## Architecture

The skin is assembled from independent XML files that Kodi merges at load time. Key layers:

- **`addon.xml`** — add-on manifest: supported resolutions (all → `xml/`), metadata, and per-language summary/description/disclaimer strings. Bump `version` and update `changelog.txt` for releases.
- **`xml/`** (104 files) — all window and dialog definitions plus shared includes:
  - `Home.xml`, `MyVideoNav.xml`, `Dialog*.xml`, etc. — one file per Kodi window/dialog (matched to Kodi by filename).
  - `Custom_1NNN_*.xml` — skin-specific custom windows/dialogs, activated by their numeric window id.
  - `View_NN_*.xml` — the container view layouts (List, Poster, IconWall, Shift, InfoWall, WideList, Wall, Banner, FanArt) referenced by media windows.
  - `Includes.xml` — **the master aggregator**: `<include file="..."/>`s every other include file and defines global `<constant>`s (e.g. `Depth*` 3D layer values, list geometry) and `<expression>`s (reusable boolean conditions). Start here to understand what's wired together.
  - `Includes_*.xml` — reusable `<include>` fragments grouped by area (Home, Buttons, MediaMenu, PVR, Games, MusicInfo, DialogSelect, Animations).
  - `Defaults.xml` — default control definitions (button, label, list, etc.) inherited skin-wide.
  - `Font.xml`, `Variables.xml` — font sets and skin variables.
- **`colors/`** — named color palettes. `defaults.xml` is the active palette; other files (`teal.xml`, `gold.xml`, `midnight.xml`, …) are alternates the user can select. Reference colors by name in XML rather than hardcoding hex.
- **`themes/curial/` and `themes/flat/`** — alternate texture sets (buttons/dialogs/lists/overlays) selectable as skin themes, compiled into `curial.xbt` / `flat.xbt`.
- **`fonts/`** — TTF font files (with license notices) referenced by `Font.xml`.
- **`language/resource.language.*/strings.po`** — localized UI strings, maintained via **Transifex** (per `changelog.txt`, most releases are just Transifex syncs). Skin-authored strings live in the `31000`–`31999` id range; do not hand-edit non-English `.po` files — they come from Transifex.
- **`playlists/*.xsp`** — smart playlists that back the home-screen widgets (recent/random/unwatched media).
- **`extras/`** — `backgrounds/` (pattern images) and `home-images/` used by the home screen.
- **`resources/`** — add-on icon, fanart, and store screenshots.

### Conventions

- Reference colors, fonts, constants, and includes **by name**; define shared values once in `colors/defaults.xml`, `Font.xml`, or `Includes.xml` rather than duplicating literals across window files.
- Put reusable markup in an `Includes_*.xml` fragment and `<include>` it; keep per-window files focused on that window's layout.
- Kodi maps most window XML files to windows **by filename** — don't rename `Dialog*.xml` / standard window files.
