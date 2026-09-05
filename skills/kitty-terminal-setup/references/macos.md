# macOS integration

Part of the `kitty-terminal-setup` skill. Verified against kitty 0.48.2 on macOS 15.

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

