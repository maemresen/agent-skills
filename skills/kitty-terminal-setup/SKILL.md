---
name: kitty-terminal-setup
description: >-
  Configure and troubleshoot the kitty terminal on macOS — config layout with an include
  split, automatic light/dark theming, background images with tint, tmux-style splits and
  tabs, and the non-obvious defaults that waste an hour if you don't know them. Use when
  setting up kitty, changing its appearance (font, colors, wallpaper, padding), adding
  split/pane or tab keybindings, making Option behave as Alt, fixing Option/Cmd+Arrow word
  and line jumps in the shell, replacing the Dock icon, or when a kitty config change
  appears to have no effect.
  Triggers: "kitty.conf", "kitty theme", "kitty background image", "kitty splits",
  "terminal wallpaper", "kitty keybindings", "option key", "alt key", "option arrow",
  "word jump", "kitty icon", "dock icon", "config change not applying".
license: MIT
metadata:
  audience: agents
  spec: agent-skills
  spec-url: https://agentskills.io
  install-as: .claude/skills/kitty-terminal-setup/
  source: https://github.com/maemresen/agent-skills
  verified-against: kitty 0.48.2 / macOS 15
---

# kitty terminal setup (macOS)

> **This is an Agent Skill, not a tutorial.** Its primary audience is a coding agent
> (Claude Code, or any [Agent Skills](https://agentskills.io)-compatible runtime), which
> reads the frontmatter `description` to decide when to load this file. Install the whole
> directory as `.claude/skills/kitty-terminal-setup/` — this file plus `references/`.
> The directory name and frontmatter do the identifying, not this file name. Humans are
> welcome; the prose is written for both.

Checked against **kitty 0.48.2** on macOS 15, against the shipped reference config and the
upstream manual. Version-specific claims are marked.

This file holds the mental model and the traps that are cheap to state. Anything with real
depth lives in `references/`, listed below — load one only when the task calls for it.

| Reference | Load it when |
|---|---|
| [`references/theming.md`](references/theming.md) | Colors, light/dark switching, or a color setting is being ignored |
| [`references/background-images.md`](references/background-images.md) | Wallpaper, tint, image layout, or the image will not change |
| [`references/keyboard.md`](references/keyboard.md) | Option/Alt behaviour, accents, or arrow keys not jumping words |
| [`references/splits-and-tabs.md`](references/splits-and-tabs.md) | Panes, tabs, layouts, or coming from tmux |
| [`references/macos.md`](references/macos.md) | Dock icon, or kitty inherited another terminal's environment |

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

Three of these have enough depth to live in `references/`:

- **`Cmd+1..9` does not switch tabs** — it is bound to splits within the current tab, and
  kitty ships no `goto_tab` bindings at all. See [`references/splits-and-tabs.md`](references/splits-and-tabs.md).
- **Option is not Alt** — `opt+…` bindings type accents until you set `macos_option_as_alt`,
  and even then Option+Arrow still will not jump words.
  See [`references/keyboard.md`](references/keyboard.md).
- **A background image that fails to load reports nothing** — no error, no warning, no log
  line. See [`references/background-images.md`](references/background-images.md).

The rest are one-liners.

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

