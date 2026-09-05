# Splits, panes, and tabs

Part of the `kitty-terminal-setup` skill. Verified against kitty 0.48.2 on macOS 15.

## `Cmd+1..9` does NOT switch tabs

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

