# Omarchy context menus

The whole Omarchy menu, as a cascading context menu.

Same tree, same entries, same `when:` guards and `✓` marks as the menu on
`SUPER + SPACE` — it reads the very same files. What changes is the
interaction: it opens under the pointer, submenus cascade sideways as you
hover, and the chain stays on screen so you can see the path you took.

Three ways in, all the same menu:

- **right-click the desktop** — where a context menu belongs
- **the bar button** — drops out of the Omarchy logo, in place of the built-in menu
- **a keybinding** — opens at the pointer, on top of whatever window has focus

Nothing hands off to the built-in menu, so you can stop using it. Applications
are listed inline with their icons, and any pane you are standing in can be
narrowed by typing at it.

## Install

```bash
omarchy plugin add https://github.com/cantinalabs/omarchy-context-menus.git --enable
```

Enabling **takes the built-in menu's place**, rather than sitting next to it.
The bar button lands in the exact slot `omarchy.menu` occupied — keeping that
entry's own settings — and the built-in menu is switched off, so there is only
ever one Omarchy glyph in your bar. Disabling puts it back where it was:

```bash
omarchy plugin disable cantina.omarchy-context-menus
```

Because it stands in for `omarchy.menu`, everything that already summoned the
built-in menu now opens this one. Your existing keybinding works untouched, as
does `omarchy-menu toggle system`, the screen-recording indicator, and anything
else routed through that id — including `omarchy-menu refresh` and
`omarchy-menu ping`.

The desktop right-click starts working immediately. No fork or clone of the
background plugin is involved, so whichever wallpaper renderer you run is left
alone.

## The keybinding

Whatever key already opened the Omarchy menu opens this one — `SUPER + SPACE`
on a stock install — with no change to your Hyprland config. It opens at the
pointer, above any window, a fullscreen one included, and closes on a second
press.

To bind another key, in `~/.config/hypr/bindings.lua`:

```lua
o.bind("SUPER + R", "Context menu", "omarchy-shell shell toggle cantina.omarchy-context-menus")
```

To land somewhere deeper in the tree, pass a route:

```lua
o.bind("SUPER + SHIFT + T", "Themes", "omarchy-shell contextMenu openRoute style.theme")
```

A route is any menu id or alias from `omarchy-menu.jsonc` — `style`, `setup`,
`apps`, `system`, `style.theme` — and the parents stay open behind it, so you
can still browse out of where you landed.

## Keys, once it is open

| | |
|---|---|
| `↑` `↓` | move within the pane |
| `→` `Enter` | open the submenu, or run the entry |
| `←` `Escape` | back one level, then close |
| type anything | narrow the pane you are standing in |
| `Backspace` | undo a character, then back one level |
| right-click | back one level, from anywhere in the menu |

Typing is what makes the Apps submenu work like the launcher it replaces:
open it, type `obs`, press `Enter`.

## Settings

The plugin reads its own entry in `~/.config/omarchy/shell.json` — the one in
`bar.layout.*` when the bar button is in use, otherwise the one in `plugins[]`.
Every key is optional. With the bar button installed they are also a form under
the bar's widget settings.

| key | default | |
|---|---|---|
| `desktopRightClick` | `true` | Right-click on bare desktop opens the menu |
| `wallpaperDoubleClick` | `true` | Double-click on bare desktop still opens the wallpaper picker |
| `inlineApps` | `true` | List applications in the menu; off sends Apps to the built-in menu |
| `submenuDelay` | `140` | Milliseconds of hover before a submenu opens |
| `paneWidth` | `268` | Pane width, in the shell's spacing units |
| `rightClickCommand` | `xdg-terminal-exec` | What right-clicking the bar button runs |

```json
{
  "bar": {
    "layout": {
      "left": [
        { "id": "cantina.omarchy-context-menus", "submenuDelay": 60 }
      ]
    }
  }
}
```

## How it fits together

The menu is not a copy of Omarchy's. It parses Omarchy's own
`default/omarchy/omarchy-menu.jsonc` and your
`~/.config/omarchy/extensions/omarchy-menu.jsonc` through a verbatim copy of
Omarchy's `MenuModel.js`, and both files are watched, so an entry you add to
the extension shows up here and in the built-in menu at the same moment,
without a restart. `when:` and `checked:` expressions are evaluated the same
way too — one batched bash process per open, not a fork per row.

Submenus whose contents only exist at runtime are filled in the same way the
built-in menu fills them: **Apps** from the shell's shared application library
(so icons, launch feedback and hidden-entry rules all match the launcher),
**Font** and **Power profile** from the same enumerations, with the current
value carrying a `✓`. Should Omarchy add a provider this plugin has not learned
yet, that one submenu — and only that one — opens the built-in menu at its
route rather than showing you an empty pane.

The desktop right-click is a transparent, screen-filling surface on the
**Bottom** layer: above the wallpaper, below every window and below the bar. It
never covers anything you can see, and it is the reason no fork of the
background plugin is needed. It does intercept the clicks that plugin would
otherwise get, so the gesture it owns is passed straight back: double-click
still opens the wallpaper picker. Its double-*right*-click theme switcher is
not, because a single right-click now opens this menu — **Style > Theme** is
two rows away.

## IPC

Everything the bar button and the catcher do is available on the
`contextMenu` target:

```bash
omarchy-shell contextMenu open                          # at the pointer
omarchy-shell contextMenu openRoute style.theme         # at the pointer, branch expanded
omarchy-shell contextMenu toggle
omarchy-shell contextMenu close
omarchy-shell contextMenu openAt HDMI-A-1 800 400       # explicit screen and position
omarchy-shell contextMenu openAtRoute HDMI-A-1 800 400 apps
omarchy-shell contextMenu openAtAnchor HDMI-A-1 168 0 24 29 below
```

`openAtAnchor` takes a rectangle and a side (`below`, `above`, `left`,
`right`) rather than a point, which is how the bar button cascades out of
itself whichever edge your bar is pinned to.

## Requirements

Omarchy 4 (the Quickshell-based `omarchy-shell`) on Hyprland. The
keybinding entry points ask Hyprland where the pointer is; everything else is
in-process.

## Licence

MIT — see [LICENSE](LICENSE). `MenuModel.js` is Omarchy's own, vendored
verbatim and used under its licence.
