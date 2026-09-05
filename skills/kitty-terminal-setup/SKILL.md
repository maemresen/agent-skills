---
name: kitty-terminal-setup
description: >-
  Configure and troubleshoot the kitty terminal on macOS — config layout with an include
  split, automatic light/dark theming, background images with tint, tmux-style splits and
  tabs, and the non-obvious defaults that waste an hour if you don't know them. Use when
  setting up kitty, changing its appearance (font, colors, wallpaper, padding), adding
  split/pane or tab keybindings, making Option behave as Alt, replacing the Dock icon, or
  when a kitty config change appears to have no effect.
  Triggers: "kitty.conf", "kitty theme", "kitty background image", "kitty splits",
  "terminal wallpaper", "kitty keybindings", "option key", "alt key", "kitty icon",
  "dock icon", "config change not applying".
license: MIT
metadata:
  audience: agents
  spec: agent-skills
  spec-url: https://agentskills.io
  install-as: .claude/skills/kitty-terminal-setup/SKILL.md
  verified-against: kitty 0.48.2 / macOS 15
---

# kitty terminal setup (macOS)

> **This is an Agent Skill, not a tutorial.** Its primary audience is a coding agent
> (Claude Code, or any [Agent Skills](https://agentskills.io)-compatible runtime), which
> reads the frontmatter `description` to decide when to load the body. Install it as
> `.claude/skills/kitty-terminal-setup/SKILL.md` — the directory name and frontmatter do
> the identifying, not this file name. Humans are welcome; the prose is written for both.


Checked against **kitty 0.48.2** on macOS 15, against the shipped reference config and the
upstream manual. Version-specific claims are marked.

## Install (macOS)

```sh
brew install --cask kitty
```

Or the vendor installer, which does not need Homebrew:

```sh
curl -L https://sw.kovidgoyal.net/kitty/installer.sh | sh /dev/stdin
```

Both put the app at `/Applications/kitty.app`. Add `/Applications/kitty.app/Contents/MacOS`
to `PATH` for the `kitty` and `kitten` binaries.

## Read the official manual first

This document does **not** restate the manual — it covers only what the manual leaves
implicit or what costs an hour to discover. For anything else, go to the source:

| Topic | Official page |
|---|---|
| Install, update, uninstall | <https://sw.kovidgoyal.net/kitty/binary/> |
| Every config option | <https://sw.kovidgoyal.net/kitty/conf/> |
| Every mappable action | <https://sw.kovidgoyal.net/kitty/actions/> |
| Layouts and splits | <https://sw.kovidgoyal.net/kitty/layouts/> |
| Windows, tabs, detaching | <https://sw.kovidgoyal.net/kitty/overview/> |
| `launch` (splits, cwd) | <https://sw.kovidgoyal.net/kitty/launch/> |
| Session files | <https://sw.kovidgoyal.net/kitty/sessions/> |
| Themes kitten | <https://sw.kovidgoyal.net/kitty/kittens/themes/> |
| FAQ | <https://sw.kovidgoyal.net/kitty/faq/> |

The best reference is the one on your own disk: `kitty.conf` as shipped is the full option
list for *your exact version*, which no web page can guarantee. `grep` it rather than
trusting memory or a blog post:

```sh
grep -n -B4 -A12 'background_tint_gaps' ~/.config/kitty/kitty.conf
```

## Config layout

Keep your settings out of the giant reference file:

```
~/.config/kitty/
├── kitty.conf                    # ~3300 lines, ALL commented — one active line:
│                                 #   include look.conf
├── look.conf                     # everything you set, EXCEPT colors
├── dark-theme.auto.conf          # colors only, dark mode
├── light-theme.auto.conf         # colors only, light mode
├── no-preference-theme.auto.conf # OS reports no preference
└── bg-<name>.png
```

`kitten edit-config` writes the fully-commented reference to `kitty.conf` if it is missing.
Keep it — it is the authoritative doc for your version.

## Automatic light/dark, and what it hijacks

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

### Populating the theme files

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

## Background image + tint

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

### `background_image_layout`: `cscaled` is CSS `cover`

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

### Picking a wallpaper

Dark, low-detail, no bright focal point. A gorgeous high-contrast sunset is unreadable behind
text. [orangci/walls-catppuccin-mocha](https://github.com/orangci/walls-catppuccin-mocha) has
336 wallpapers recolored to the Catppuccin palette, matching a Catppuccin theme exactly.

### Reload re-reads the config, not the image

A reload re-parses config *files*. It does **not** re-read a file the config points at.
Overwrite `bg.png` in place and kitty keeps showing the cached image forever, with no error.

**Fix:** change the *path*, not the bytes. Name each wallpaper distinctly
(`bg-cottages-river.png`) and repoint `background_image`.

## Font

```conf
font_family      JetBrainsMono Nerd Font Mono
font_size        14.0
disable_ligatures cursor
modify_font cell_height 105%
macos_thicken_font 0.3
```

Family is preference; the rule that matters is the **`Mono`** Nerd Font variant. The
non-Mono variant renders icons double-width and misaligns TUI box drawing (Claude Code, k9s,
lazygit). `disable_ligatures cursor` keeps ligatures but unfolds the one under the cursor.

Install and *verify* — kitty falls back silently to another font with only a stderr warning,
and you will blame the wrong thing:

```sh
brew install --cask font-jetbrains-mono-nerd-font
kitten list-fonts | grep -i 'nerd font mono'
```

`font_size` defaults to 11.0, which is small on a Retina display.

## Reloading

**You almost never need to reload.** `auto_reload_config` is on by default and kitty watches
its config files, so saving `look.conf` applies within a tenth of a second. Editing a config
in one pane and watching a second pane change live is the fastest way to tune padding, tint or
font size.

| Method | Notes |
|---|---|
| Nothing | `auto_reload_config` is on by default — kitty watches and reloads on save |
| `Ctrl+Cmd+,` | `load_config_file` (macOS default; `Ctrl+Shift+F5` elsewhere) |
| `kill -SIGUSR1 $KITTY_PID` | No setup, scriptable |
| `kitten @ load-config` | Requires `allow_remote_control yes` first |

Three traps in the option itself:

```conf
auto_reload_config 0.1
```

- **It is a number of seconds, not a boolean.** It is a debounce interval so a burst of saves
  triggers one reload. `auto_reload_config yes` is invalid; a *negative* value is what
  disables it.
- **It is read only at startup.** Upstream: "Changes to this setting by reloading
  configuration are ignored." Setting it in a running kitty does nothing until restart.
- **It needs `kitty.conf` to have existed when kitty started.** Create the file first, then
  launch, or watching silently does not happen.

If colors snap back to `kitty.conf` values on reload while using auto light/dark, that is a
known bug in older versions — upgrade kitty.

After any change to `background_*`, actually look at the terminal. A background image that
fails to load produces no error, no warning, and no log line — just no image.

## Defaults that surprise people

### `Cmd+1..9` does NOT switch tabs

It is bound to `first_window` / `second_window` — **splits within the current tab**. With one
window per tab the keys look dead. kitty ships **no** `goto_tab` bindings at all:

```conf
map cmd+1 goto_tab 1
map cmd+2 goto_tab 2
map cmd+3 goto_tab 3
map cmd+4 goto_tab 4
map cmd+5 goto_tab 5
map cmd+6 goto_tab 6
map cmd+7 goto_tab 7
map cmd+8 goto_tab 8
map cmd+9 goto_tab -1
```

Pair it with a template so you can tell tabs apart, and a bar that is legible:

```conf
tab_bar_edge          top
tab_bar_style         powerline
tab_powerline_style   slanted
tab_bar_min_tabs      2
tab_title_template    "{index}: {title}"
```

### Option is NOT Alt — `opt+…` bindings type accents instead

Default `macos_option_as_alt no` lets macOS handle Option+Key **first** as Unicode entry.
So `Option+I` is the dead key for `ˆ`, `Option+E` for `´`, `Option+N` for `˜`, `Option+U`
for `¨` — the character is typed *before* your `map opt+i …` binding ever fires, and the
binding looks dead only on those letters. Same reason `Alt+B` / `Alt+F` word jumps and any
`Alt+Key` shortcut inside a TUI do nothing.

```conf
macos_option_as_alt yes
```

- **Cost:** the native accent dead keys are gone. Use `ctrl+shift+u` (`unicode_input`) or
  the macOS character picker (`Ctrl+Cmd+Space`) instead.
- **Halfway:** `left` (or `right`) makes only one Option key an Alt and keeps the other for
  accents.
- **Needs a restart.** Upstream: "Changing this option by reloading the config is not
  supported." Save, then fully quit (`Cmd+Q`) and relaunch — a reload silently ignores it.
- kitty's **own** shortcuts always treat Option as Alt regardless of this setting, and an
  `opt+…` map wins over the program inside the window: bind `opt+j` and no TUI will ever
  see `Alt+J`.

### `macos_option_as_alt yes` still leaves Option+Arrow dead

Turning the option on fixes `Alt+`*letter*. It does **not** give you Option+Arrow word
jumps, and nothing tells you why.

Option+Arrow emits `ESC [ 1 ; 3 D` / `ESC [ 1 ; 3 C`. zsh and bash bind `ESC b` / `ESC f`
to `backward-word` / `forward-word` — not the arrow sequence, which is unbound. iTerm2
ships a default key mapping that rewrites `⌥←` / `⌥→` into `ESC b` / `ESC f`, so the keys
appear to "just work" there. kitty ships no such table. Map it yourself:

```conf
map opt+left  send_text all \x1bb
map opt+right send_text all \x1bf
map cmd+left  send_text all \x01
map cmd+right send_text all \x05
```

`\x01` / `\x05` are `Ctrl+A` / `Ctrl+E`, which zsh and bash bind to line start / line end.

**Cost, and it is real:** `send_text all` fires in *every* program, not just the shell.
`\x01` is the default tmux prefix and vim's increment; `\x1bb` replaces whatever `Alt+B`
did inside a TUI. Scope it with `send_text normal` if that bites, or skip the maps and
bind the sequences shell-side instead (`bindkey '^[[1;3D' backward-word`), which leaves
TUIs alone but only fixes the one shell.

To see what a key actually sends, run `kitten show-key` and press it. That prints the
bytes the program receives, which is the only reliable way to settle this — `bindkey`
alone tells you what the shell listens for, not what kitty sends it.

### No trailing comments on `map` lines

```conf
map cmd+shift+m detach_window ask   # BROKEN — parsed as action arguments
```

kitty has no trailing-comment syntax. Comments go on their own line.

### No rounded corners

There is no radius option for the terminal text area — it is drawn as a rectangle. The only
matches in the reference config are `scrollbar_radius` and `tab_powerline_style round`. Your
window gets macOS's native rounded corners; that's the lot.

### Rename a tab

`Shift+Cmd+I` — already a default, no config needed.

### `kitten icat` needs a TTY

It fails with `open /dev/tty: device not configured` when run from anything without a
controlling terminal (an agent shell, a subprocess). Use `open -a Preview` there instead.

## tmux-style splits

kitty calls them *windows*. The `splits` layout gives arbitrary nesting.

```conf
enabled_layouts splits:split_axis=horizontal,stack

# create
map cmd+d       launch --location=vsplit --cwd=current
map cmd+shift+d launch --location=hsplit --cwd=current

# navigate (IJKL is one ergonomic choice; arrows below are the other)
map opt+i neighboring_window up
map opt+j neighboring_window left
map opt+k neighboring_window down
map opt+l neighboring_window right
map cmd+opt+up    neighboring_window up
map cmd+opt+down  neighboring_window down
map cmd+opt+left  neighboring_window left
map cmd+opt+right neighboring_window right

# move a pane around the tree
map opt+shift+i move_window up
map opt+shift+j move_window left
map opt+shift+k move_window down
map opt+shift+l move_window right
map cmd+shift+left  move_window_backward
map cmd+shift+right move_window_forward

# resize
map cmd+ctrl+left  resize_window narrower
map cmd+ctrl+right resize_window wider
map cmd+ctrl+up    resize_window taller
map cmd+ctrl+down  resize_window shorter

# arrange
map cmd+shift+z toggle_layout stack
map cmd+shift+r layout_action rotate
map cmd+shift+e layout_action equalize
map cmd+shift+b layout_action bias 75
```

`enabled_layouts` defaults to every layout **in alphabetical order**, so the startup layout
is `fat`, not `splits`. Set it explicitly.

`--cwd=current` uses the source window's working directory and needs nothing. (It is
`--cwd=last_reported` that requires shell integration — easy to conflate.)

| tmux | kitty | source |
|---|---|---|
| `prefix %` / `"` | `Cmd+D` / `Cmd+Shift+D` | configured above |
| `prefix ←↑↓→` | `Opt+I/J/K/L` or `Cmd+Opt+arrows` | configured above |
| `prefix z` | `Cmd+Shift+Z` | configured above |
| `prefix {` / `}` | `Cmd+Shift+←` / `Cmd+Shift+→` | configured above |
| `prefix Ctrl+arrows` (resize) | `Cmd+Ctrl+arrows` | configured above |
| `prefix !` (break pane) | `Cmd+Shift+N` | configured below |
| `prefix 0-9` | `Cmd+1..9` | configured above |
| `prefix ,` (rename) | `Shift+Cmd+I` | **kitty default** |
| `prefix Space` | `Ctrl+Shift+L` (next layout) | **kitty default** |
| — | `Cmd+Shift+R` rotate split axis | no tmux equivalent |

With `enabled_layouts splits,stack` there are only two layouts, so `Ctrl+Shift+L` and
`Cmd+Shift+Z` do much the same thing.

### `layout_action rotate` is per-split

It rotates the split containing the **focused** pane and its sibling, not the whole tab. To
turn a `left | (top/bottom)` arrangement into three columns, focus a right-hand pane first,
then `Cmd+Shift+R`, then `Cmd+Shift+E`.

### No `join-pane` equivalent

tmux's `join-pane -h -s <src> -t <dst>` moves a pane *and* sets its orientation in one
command. kitty splits this in two:

```conf
map cmd+shift+m detach_window ask
map cmd+shift+, detach_window tab-left
map cmd+shift+. detach_window tab-right
map cmd+shift+n detach_window new-tab
map cmd+shift+o detach_window
```

`detach_window` picks a destination tab only — never the axis or position. Move first, then
rearrange with `rotate` / `move_window`.

### kitty splits are not a tmux replacement

**No detach/reattach.** Panes die with the window; there is no `tmux attach` after closing
the terminal or dropping SSH. If that is why you use tmux, keep tmux.

For a fixed arrangement, use a session file instead of rebuilding by hand:

```
new_tab work
layout splits
launch --cwd=/path/to/project
launch --location=vsplit
launch --location=vsplit
```

`kitty --session ~/.config/kitty/work.session`, or wire it to `startup_session`.

## Other settings worth setting

Non-default, small, and they make daily use better:

```conf
cursor_trail          3       # motion trail; default 0
cursor_blink_interval 0       # stop the blink
inactive_text_alpha   0.85    # dim unfocused splits — the only focus cue you get
scrollback_lines      20000   # default 2000
enable_audio_bell     no      # visual bell instead
window_alert_on_bell  yes
```

## Custom app icon (macOS)

Drop a `kitty.app.png` (or `kitty.app.icns`) into the config directory and kitty applies it
at startup — no setting, no `map`, no rebuild of the app bundle. A 512×512 PNG is enough.

```sh
cp my-icon.png ~/.config/kitty/kitty.app.png
```

Two things are not obvious:

- **It is applied at startup only.** A config reload does not pick up a new or changed
  icon — fully quit (`Cmd+Q`) and relaunch.
- **The Dock keeps its own cache and reverts the icon when kitty quits.** The change is real
  (the running app shows it — check Cmd+Tab), but the Dock tile shows the stock icon again
  after the next quit until the cache is flushed:

  ```sh
  rm /var/folders/*/*/*/com.apple.dock.iconcache; killall Dock
  ```

If you would rather not keep the file in the config folder, the FAQ documents a one-shot
`kitty +runpy … cocoa_set_app_icon …` command, but it does not survive a relaunch. The file
is the durable way.

## Environment inheritance gotcha

If kitty is exec'd from a shell rather than started clean, it inherits that shell's entire
environment and hands it to every window it opens — including another terminal's markers
(`TERM_PROGRAM`, and that terminal's own variables). Tools that detect the terminal from
`TERM_PROGRAM` are then wrong. A `SHLVL` in the app's own environment is the tell:

```sh
ps eww -p $KITTY_PID | tr ' ' '\n' | grep -E '^(TERM_PROGRAM|SHLVL)='
```

A Dock/Spotlight launch has no `SHLVL` at all. Fully quit (`Cmd+Q` — a new tab inherits the
same app env) and relaunch, or `open -a kitty`, which asks launchd to start it with a clean
environment even when typed inside another terminal.

