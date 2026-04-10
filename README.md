# tmux-claude

tmux integration for Claude Code. Shows a status bar indicator when any Claude
Code session is waiting for input, and provides a keybinding to jump to it.

## Features

- **Status bar indicator**: `[Claude waiting]` (or `[Claude: N waiting]`) appears
  in your tmux status bar when Claude sessions need attention
- **Jump to waiting pane**: `prefix + g` switches to the oldest waiting Claude
  pane. If you're already in a waiting pane, it clears that pane's waiting state
  and advances to the next one.
- **Bell on wait**: a bell character is sent so tmux highlights the window immediately
- **Auto-clear on focus**: switching to a pane clears its waiting state
- **Zero config**: no changes to `tmux.conf` required — everything is configured
  programmatically
- **Safe outside tmux**: all hooks exit silently when not in a tmux session, so
  Claude Code works normally without tmux

## Install

```sh
git clone <repo-url> ~/src/tmux-claude
~/src/tmux-claude/bin/tmux-claude-setup
```

Run `tmux-claude-setup` from anywhere — inside or outside tmux. It registers
Claude Code hooks in `~/.claude/settings.json` (merged with your existing hooks,
not overwritten). The tmux-side configuration (status bar, keybinding, focus
hooks) is set up automatically the first time a Claude Code hook fires inside a
tmux session, so there's nothing else to do.

Setup is idempotent — safe to run multiple times.

## Uninstall

```sh
~/src/tmux-claude/bin/tmux-claude-teardown
```

This surgically removes only what was added: hooks are filtered out of
`settings.json`, the status bar segment is stripped from the live `status-right`
value, and tmux keybindings/hooks are unregistered.

## How it works

Claude Code hooks write state files to `/tmp/tmux-claude-<uid>/`:

| Event | Action |
|---|---|
| `Stop`, `Notification`, `PermissionRequest` | Create `<pane-id>.waiting` + send bell |
| `UserPromptSubmit`, `SessionEnd` | Remove `<pane-id>.waiting` |
| tmux pane/window focus | Remove `<pane-id>.waiting` |

The status bar script reads this directory and outputs the indicator. The jump
script finds the oldest `.waiting` file and switches to that pane.

## Scripts

| Script | Purpose |
|---|---|
| `tmux-claude-setup` | One-time install: register Claude Code hooks |
| `tmux-claude-teardown` | Clean uninstall of everything |
| `tmux-claude-ensure-tmux` | Lazy init: configure tmux on first hook fire |
| `tmux-claude-notify` | Hook: create waiting state + bell |
| `tmux-claude-resume` | Hook: clear waiting state (prompt submit) |
| `tmux-claude-cleanup` | Hook: clear waiting state (session end) |
| `tmux-claude-focus` | tmux hook: clear waiting state (pane focus) |
| `tmux-claude-status` | Status bar: count waiting panes, output indicator |
| `tmux-claude-jump` | Keybinding: switch to oldest waiting pane |

## Requirements

- tmux 3.x+
- python3 (for JSON manipulation during setup/teardown)
- Claude Code with hooks support
