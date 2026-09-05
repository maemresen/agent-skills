# Option, Alt, and arrow-key traps

Part of the `kitty-terminal-setup` skill. Verified against kitty 0.48.2 on macOS 15.

## Option is NOT Alt — `opt+…` bindings type accents instead

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

## `macos_option_as_alt yes` still leaves Option+Arrow dead

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

