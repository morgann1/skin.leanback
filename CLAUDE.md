# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **skin.leanback**, a custom Kodi skin forked from Estuary. It targets 1920x1080 (16:9) as the default resolution. The skin is authored by morgann1.

## Design Spec Scaling

The Figma design spec is at **960x540**. All pixel values from the design must be **multiplied by 2x** when implementing in the skin XML.

## Architecture

### XML Include System

`xml/Includes.xml` is the root orchestrator that chains all other include files:
- `Defaults.xml` — default control properties
- `Includes_Home.xml` — home screen widget layouts (WidgetListPoster, WidgetListEpisodes, etc.)
- `Includes_Buttons.xml` — button components (NavButton, NavIconButton, OSDButton, etc.)
- `Includes_Animations.xml` — shared animation definitions
- `Variables.xml` — dynamic content variables and expressions

Includes are parameterized with `<param>` tags for flexible reuse.

### Navigation State (Home Screen)

The home screen uses `Window(Home).Property(nav_id)` to track which nav tab is active. Each NavButton sets this property via `<onfocus>SetProperty(nav_id,<value>,Home)</onfocus>`. Content sections use `String.IsEqual(Window(Home).Property(nav_id),<id>)` for visibility.

Navigation between navbar buttons uses explicit `onleft`/`onright`/`ondown` with `SetFocus(<id>)` rather than relying on grouplist auto-navigation.

### Depth Layering

Controls use depth constants for z-ordering: `DepthBackground` (-0.80) → `DepthContentPanel` (0.05) → `DepthBars` (0.20) → `DepthOSD` (0.40) → `DepthDialog` (0.50) → `DepthMax` (0.54).

### Color System

`colors/defaults.xml` defines all colors in ARGB hex format (e.g., `FF1A1C1E`). Key colors:
- `primary_background`: `FF1A1C1E` (dark neutral)
- `button_focus`: `FF12A0C7`
- Text uses inline hex colors: `FFE3E2E6` (light), `FF1A1C1E` (dark, for focused states)

### Font System

`xml/Font.xml` defines two fontsets (Default using Inter, fallback using Arial). Custom fonts:
- `font28` — Inter-Medium.ttf at 28px (used for nav buttons, maps to 14px Figma)
- Font names follow the pattern `font<size>` or `font<size>_title`

### Texture System

- Compiled `.xbt` bundles in `media/` contain most textures
- Loose files in `media/` (like `rounded_rect.png`, `rounded_mask.png`) are resolved by Kodi relative to the `media/` directory
- Icon assets in `media/icons/` should be **white on transparent** PNG for `colordiffuse` tinting
- `extras/` contains background images referenced via `special://skin/extras/`

### Rounded Corners

Kodi has no native border-radius. Two techniques are used:
- **9-slice rounded rect** (`rounded_rect.png`): white rounded rectangle, tinted via `colordiffuse`, used with `border="8"` for button backgrounds
- **Corner mask** (`rounded_mask.png`): background-colored frame with transparent center, overlaid on content to fake rounded corners

## Key Component Patterns

### NavButton (text navigation button)
Defined in `Includes_Buttons.xml`. A `<button>` control with params: `id`, `width`, `height`, `label`, `nav_id`, `onclick`, `ondown`, `onleft`, `onright`.

### NavIconButton (icon navigation button)
Uses `<radiobutton>` with radio textures for the icon, allowing different `colordiffuse` per focus state. Icon tints: `FFE3E2E6` unfocused, `FF1A1C1E` focused.

## File Conventions

- Window definitions: `xml/Home.xml`, `xml/MyVideoNav.xml`, `xml/Settings.xml`, etc.
- Dialog definitions: `xml/Dialog*.xml`
- Custom windows: `xml/Custom_1100-1110_*.xml`
- Views: `xml/View_50-55_*.xml`, `xml/View_500-503_*.xml`
- Localization: `language/resource.language.<locale>/strings.po`

## Skin Reload

A keymap at `%APPDATA%\Kodi\userdata\keymaps\reload_skin.xml` binds **F5** to `ReloadSkin()`.
