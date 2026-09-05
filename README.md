# agent-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Personal [Agent Skills](https://agentskills.io) for the developer environment — the
non-obvious parts of terminals and shells that cost an hour to discover.

Tool-neutral by design: each skill is a plain `SKILL.md` that any Agent Skills-compatible
runtime can load. Packaged as a Claude Code plugin for convenience, not as a dependency.

## Skills

| Skill | What it covers |
|---|---|
| [`kitty-terminal-setup`](skills/kitty-terminal-setup/SKILL.md) | kitty on macOS — config layout, automatic light/dark theming, background images and tint, tmux-style splits, and the defaults that waste an hour |

## Install as a Claude Code plugin

```sh
/plugin marketplace add maemresen/agent-skills
/plugin install personal-ai@maemresen
```

## Install a single skill, any runtime

No plugin needed. A skill is a directory: `SKILL.md` plus a `references/` folder the
runtime loads on demand, so copy the whole thing rather than the one file.

```sh
git clone --depth 1 https://github.com/maemresen/agent-skills /tmp/agent-skills
cp -R /tmp/agent-skills/skills/kitty-terminal-setup ~/.claude/skills/
```

## Layout

```
skills/<skill-name>/
  SKILL.md          # frontmatter + mental model + cheap-to-state traps
  references/       # depth, loaded only when the task calls for it
```

`SKILL.md` stays small on purpose. The frontmatter `description` is the routing surface a
runtime reads to decide whether to load the skill at all, and everything with real depth
sits behind a relative link so it costs nothing until it is needed.

## Scope

Skills here are about **method** — how a tool behaves and where it surprises you. They carry
no personal data and no organisation-specific configuration, so they are safe to install
anywhere, including a work machine.

Actual dotfiles live elsewhere: [`yazilim-vip/terminal-setup`](https://github.com/yazilim-vip/terminal-setup)
holds the Ghostty and tmux configs these skills describe how to reason about.

## Licence

MIT — see [LICENSE](LICENSE).
