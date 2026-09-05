# Background image + tint

Part of the `kitty-terminal-setup` skill. Verified against kitty 0.48.2 on macOS 15.

In `look.conf` (**not** the theme files — see the section above):

```conf
background_image        ~/.config/kitty/bg-<name>.png
background_image_layout cscaled
background_tint         0.90
background_tint_gaps    1.0
```

- `background_tint` — blends the image toward the **theme background color**. `0` = raw
  image, `1` = solid color. `0.85`–`0.92` keeps text readable over a detailed image.
- `background_tint_gaps` — **multiplicative** with `background_tint`, default `1.0`, applied
  to padding and margins. This is the one people get backwards:

| Value | Result |
|---|---|
| `1.0` | Gaps tinted like the text area → padding is **inside** the dark panel (breathing room around text) |
| `0.0` | Gaps show the **raw image** → dark panel hugs the text exactly, wallpaper as a border |

Negative values are not documented; don't copy them from dotfiles.

kitty does not distinguish margin from padding for tinting — both are "gaps". You cannot have
tinted padding *and* a wallpaper-visible margin.

`tint_gaps` only does something if there are gaps, so set them in `look.conf`:

```conf
window_padding_width 14 16
window_margin_width  2
```

## `background_image_layout`: `cscaled` is CSS `cover`

The value names give no clue what they do. Only one of them fills the window, keeps the
aspect ratio, and crops the overflow centered — `cscaled`.

| Value | Behaviour |
|---|---|
| `cscaled` | **Cover** — fill the window, preserve aspect ratio, crop centered |
| `scaled` | Fill the window, **ignore** aspect ratio (stretches) |
| `centered` | Original pixel size, centered, no scaling |
| `clamped` | Anchored top-left, no scaling |
| `tiled` / `mirror-tiled` | Repeat |

From the renderer (`draw_bg_image` in `shaders.c`): for `cscaled`, when the image is wider
than the window it scales the image to the window height and lets it overflow horizontally,
then trims `(vwidth - iwidth) / vwidth` off the left and right edges symmetrically. It
recomputes on every resize, so the image stays centered and correctly cropped at any window
size.

Two things this implies:

- **There is no "contain" / letterbox option.** Nothing fits the whole image inside the
  window with bars.
- **The crop cannot be biased off-center.** The trim is symmetric with no offset parameter,
  so if you want a different part of a wide image visible, crop the file itself.

## Picking a wallpaper

Dark, low-detail, no bright focal point. A gorgeous high-contrast sunset is unreadable behind
text. [orangci/walls-catppuccin-mocha](https://github.com/orangci/walls-catppuccin-mocha) has
336 wallpapers recolored to the Catppuccin palette, matching a Catppuccin theme exactly.

## Reload re-reads the config, not the image

A reload re-parses config *files*. It does **not** re-read a file the config points at.
Overwrite `bg.png` in place and kitty keeps showing the cached image forever, with no error.

**Fix:** change the *path*, not the bytes. Name each wallpaper distinctly
(`bg-cottages-river.png`) and repoint `background_image`.

