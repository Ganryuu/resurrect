# resurrect

Sometimes my battery dies, or I close a terminal window by mistake, and I lose my whole
setup: the windows, how they were organized, and the agents running in them. `resurrect`
gets it all back with one command:

```sh
resurrect restore
```

Every terminal tab and zellij session reopens, with each coding agent back on the conversation
it had open:

| Agent       | Reopened with              |
|-------------|----------------------------|
| Claude Code | `claude --resume <id>`     |
| Codex       | `codex resume <id>`        |
| OpenCode    | `opencode --session <id>`  |

One Python file, no dependencies.

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/ganryuu/resurrect/main/resurrect -o ~/.local/bin/resurrect
chmod +x ~/.local/bin/resurrect
resurrect autosave on
```

## Use

```
resurrect restore [N] [--dry-run]   reopen what was open and isn't now
resurrect save                      snapshot now
resurrect show [N]                  the live snapshot and the history, or all of snapshot N
resurrect autosave on|off           snapshot every minute (systemd user timer)
```

## How it works

Every minute it records:

- each gnome-terminal tab: its folder, and whether it runs an agent or a zellij client
- each zellij session's layout (`zellij action dump-layout`), with each agent pane tied to its conversation
- which conversation each agent has open:
  - Claude Code: `~/.claude/sessions/<pid>.json`
  - Codex: the rollout file it has open, or `codex resume <id>` on its command line, or else the
    newest terminal thread in `~/.codex/sessions` written in its folder since it started
  - OpenCode: `--session <id>` on its command line, or else the newest session in
    `~/.local/share/opencode/opencode.db` updated in its folder since it started

Once those terminals are gone (dead battery, reboot, logout), that snapshot moves into the
history, so the terminals you open afterwards can't overwrite it. `restore` opens one window
with a tab per saved tab, rebuilds the zellij sessions from their layouts and resumes each agent
wherever it was. An agent that hadn't started a conversation comes back fresh.
Anything still running is left alone, so running it twice is safe.

The history keeps the last 20 snapshots, one for each time the terminals went down; `show`
numbers them, newest first. `restore` takes the one not restored yet and asks which if there
are several; `restore N` reopens any of them. A terminal that only ever held empty shells in
your home folder leaves nothing behind.

State lives in `~/.local/state/resurrect/`.

## Requirements

Linux with systemd, Python 3.8+, gnome-terminal and `ss` (iproute2). Tested with zellij 0.44,
Claude Code 2.1, Codex 0.163 and OpenCode 1.18; all are optional.

## Limits

- All tabs reopen in one window: gnome-terminal doesn't expose which tab was in which window.
- Shell tabs come back in their folder but don't re-run what was running in them. Other
  commands in zellij panes wait for Enter, as zellij does after a crash.
- Two agent panes of the same kind in the same folder and the same zellij tab are matched in
  screen order, so they can swap places.
- Codex (with its background server) and OpenCode don't say which conversation a terminal shows,
  so it's inferred from what was written in that folder since it started. Two of them open in
  the same folder can swap conversations, and one that resumed a conversation and hasn't
  written to it since comes back fresh.
- The session files of all three agents are internals and may change between versions.

## License

MIT
