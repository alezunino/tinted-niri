# tinted-niri

This repository is meant to work with [tinted-theming/home](https://github.com/tinted-theming/home). It provides templates that can be used with base16 and base24 color schemes to generate functional theme snippets for [Niri](https://github.com/YaLTeR/niri), a scrollable-tiling Wayland compositor.

Modeled after [tinted-theming/base16-i3](https://github.com/tinted-theming/base16-i3).

## What this themes

Each file in `themes/` is a KDL snippet setting only Niri's color properties:

- `layout { focus-ring { active-color base0B, inactive-color base03, urgent-color base08 } }`
- `layout { border { width 3, active-color base0B, inactive-color base03, urgent-color base08 } }`
- `window-rule { match is-window-cast-target=true ... }` for screencast targets:
  - `focus-ring { active-color base0E, inactive-color base08 }`
  - `border { on, inactive-color base08 }`
  - `shadow { color base08F0 }`
  - `tab-indicator { active-color base0E, inactive-color base08 }`

Example (`themes/base16-default-dark.kdl`):

```kdl
layout {
    focus-ring {
        active-color "#a1b56c"
        inactive-color "#585858"
        urgent-color "#ab4642"
    }

    border {
        width 3
        active-color "#a1b56c"
        inactive-color "#585858"
        urgent-color "#ab4642"
    }
}

window-rule {
    match is-window-cast-target=true

    focus-ring {
        active-color "#ba8baf"
        inactive-color "#ab4642"
    }

    border {
        on
        inactive-color "#ab4642"
    }

    shadow {
        color "#ab4642F0"
    }

    tab-indicator {
        active-color "#ba8baf"
        inactive-color "#ab4642"
    }
}
```

All other Niri settings (gaps, widths, binds, outputs, etc.) stay in your own config. Niri merges included files per-property, so only the colors above are overridden.

## Requirements

- Niri with top-level `include` support (v25.11+)
- [tinty](https://github.com/tinted-theming/tinty) (recommended)

## Usage (recommended: tinty)

1. Install and sync tinty:

   ```sh
   tinty sync
   ```

2. Add this item to `~/.config/tinted-theming/tinty/config.toml`:

   ```toml
   [[items]]
   path = "https://github.com/alezunino/tinted-niri"
   name = "tinted-niri"
   themes-dir = "themes"
   hook = "cp -f \"$TINTY_THEME_FILE_PATH\" ~/.config/niri/tinted-niri.kdl"
   supported-systems = ["base16", "base24"]
   ```

   Then sync again:

   ```sh
   tinty sync
   ```

3. Include the theme file from your Niri config. In `~/.config/niri/config.kdl`, at the top level (not inside any other block), and after your own `layout { }` block so the theme takes precedence:

   ```kdl
   include "~/.config/niri/tinted-niri.kdl"
   ```

   Includes are positional: settings in the included file override settings defined prior to the `include` line, and can themselves be overridden by settings defined after it. Included files are watched, so Niri live-reloads when tinty rewrites the file — no restart needed.

4. Apply a scheme:

   ```sh
   tinty apply base16-default-dark
   tinty apply base24-tokyo-night-dark
   ```

   Validate your config if anything looks off:

   ```sh
   niri validate
   ```

## Usage (manual, without tinty)

Download any pre-generated theme and include it:

```sh
curl https://raw.githubusercontent.com/alezunino/tinted-niri/main/themes/base16-default-dark.kdl > ~/.config/niri/tinted-niri.kdl
```

```kdl
// ~/.config/niri/config.kdl, top level
include "~/.config/niri/tinted-niri.kdl"
```

To try another scheme, just overwrite `tinted-niri.kdl`:

```sh
curl https://raw.githubusercontent.com/alezunino/tinted-niri/main/themes/base16-catppuccin-mocha.kdl > ~/.config/niri/tinted-niri.kdl
```

Niri picks up the change automatically.

If you want Niri to start even when the theme file is missing, use an optional include:

```kdl
include optional=true "~/.config/niri/tinted-niri.kdl"
```

## Templates

Defined in `templates/config.yaml`:

```yaml
base16-default:
  extension: .kdl
  filename: "themes/{{ scheme-system }}-{{ scheme-slug }}.kdl"

base24-default:
  extension: .kdl
  filename: "themes/{{ scheme-system }}-{{ scheme-slug }}.kdl"
  supported-systems: [base24]
```

- `templates/base16-default.mustache`: base16 template
- `templates/base24-default.mustache`: base24 template

## Themes

`themes/` contains pre-generated `.kdl` files for every scheme, named `base16-<slug>.kdl` and `base24-<slug>.kdl` (e.g. `base16-default-dark.kdl`, `base24-tokyo-night-dark.kdl`).

They are built with [tinted-theming/home](https://github.com/tinted-theming/home) from the templates above.

## Customization

Because Niri merges includes per-property, keep your layout details in `config.kdl` and let the theme supply only colors:

```kdl
layout {
    gaps 8
    border {
        width 2
    }
}

// Theme colors win for focus-ring / border colors defined in the theme.
include "~/.config/niri/tinted-niri.kdl"

// Anything after the include wins over the theme, e.g. force your own width:
// layout {
//     border {
//         width 2
//     }
// }
```

Note the border special case from the Niri docs: writing `layout { border {} }` with no properties in an included file does nothing. This template always sets explicit colors (and `width 3` / `on` where needed), so it applies correctly as an include.

## Contributing

To regenerate themes, use the tinted-theming builder per [tinted-theming/home](https://github.com/tinted-theming/home). Do not edit files in `themes/` by hand — edit the `.mustache` templates instead.

## License

MIT — see [LICENSE](LICENSE).
