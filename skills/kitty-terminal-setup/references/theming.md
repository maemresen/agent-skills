# Automatic light/dark, and what it hijacks

Part of the `kitty-terminal-setup` skill. Verified against kitty 0.48.2 on macOS 15.

kitty auto-loads `dark-theme.auto.conf` / `light-theme.auto.conf` from the config dir when
the OS appearance changes. No `include` needed, no restart. This is `auto_color_scheme`.

**These files have the highest precedence there is.** Upstream, verbatim:

> Note that the colors in these files override all other colors, and also all background
> image settings, even those specified using the `kitty --override` command line flag.

Two consequences people learn the hard way:

1. **Colors in `kitty.conf` or an included file are silently ignored.** Not overridden with a
   warning — ignored. Put colors only in the auto files.
2. **`background_*` is where the docs and reality disagree — trust reality.** The reference
   config claims, for each of `background_image`, `background_image_layout`,
   `background_image_linear`, `background_tint`, `background_tint_gaps` and
   `macos_titlebar_color`: *"when using auto_color_scheme this option is overridden by the
   color scheme file and must be set inside it to take effect."*

   On 0.48.2 that is **not** what happens. These settings work from `kitty.conf` / an
   included file, and moving them into the `*-theme.auto.conf` files makes the background
   silently disappear. (Likely cause, unconfirmed: `~` is not expanded in the auto files, so
   `background_image` resolves to a literal path that does not exist and renders nothing.)

   **Keep colors in the theme files and `background_*` in your main config.** If you do move
   them, use an absolute path and verify before you trust it — kitty reports nothing when a
   background image fails to load.

## Populating the theme files

Use the themes kitten in auto mode; it writes exactly these files:

```sh
kitten themes --dump-theme "Catppuccin-Mocha" > ~/.config/kitty/dark-theme.auto.conf
kitten themes --dump-theme "Catppuccin-Latte" > ~/.config/kitty/light-theme.auto.conf
echo 'include dark-theme.auto.conf' > ~/.config/kitty/no-preference-theme.auto.conf
```

`no-preference-theme.auto.conf` is used when the OS reports no light/dark preference; a
one-line `include` of your dark theme is the tidy answer.

**Trap:** plain interactive `kitten themes` (without the dark/light auto flow) does something
different — it writes `current-theme.conf` and appends `include current-theme.conf` to
`kitty.conf`, then comments out your existing color settings. That is a *manual* theme, and
it puts colors in an included file, which the auto files will then override. Pick one model.

