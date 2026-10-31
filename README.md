# resurrect

The laptop died in your backpack. `resurrect restore` brings back every gnome-terminal tab,
every zellij session with its layout, and every Claude Code conversation that was running in
them, resumed in the same folder with `claude --resume`.

One Python file, no dependencies.

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/ganryuu/resurrect/main/resurrect -o ~/.local/bin/resurrect
chmod +x ~/.local/bin/resurrect
resurrect autosave on
```

## Use

```
resurrect restore [--dry-run]   reopen what was open and isn't now
resurrect save                  snapshot now
resurrect show                  what the snapshot holds
resurrect autosave on|off       snapshot every minute (systemd user timer)
```

## How it works

Every minute it records:

- each gnome-terminal tab: its folder, and whether it runs Claude Code or a zellij client
- each zellij session's layout (`zellij action dump-layout`), with each claude pane tied to its conversation
- which Claude Code session runs where, from `~/.claude/sessions/<pid>.json`

Once those terminals are gone (dead battery, reboot, logout), that snapshot is set aside so
later saves can't overwrite it. `restore` opens one window with a tab per saved tab, rebuilds
the zellij sessions from their layouts and runs `claude --resume <id>` wherever Claude was.
Anything still running is left alone, so running it twice is safe.

State lives in `~/.local/state/resurrect/`.

## Requirements

Linux with systemd, Python 3.8+, gnome-terminal and `ss` (iproute2). Tested with zellij 0.44
and Claude Code 2.1; both are optional.

## Limits

- All tabs reopen in one window: gnome-terminal doesn't expose which tab was in which window.
- Shell tabs come back in their folder but don't re-run what was running in them. Other
  commands in zellij panes wait for Enter, as zellij does after a crash.
- Two Claude panes in the same folder and the same zellij tab are matched in screen order, so
  they can swap places.
- `~/.claude/sessions` is a Claude Code internal and may change between versions.

## License

MIT
