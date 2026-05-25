## [0.1.3] - 2026-05-24

### Fixed
- **Hooks now install into `~/.claude/settings.json` again.** The 0.1.2 move to
  `~/.claude/settings.local.json` was a regression: Claude Code does not load a
  user-level `settings.local.json` (the `.local.json` variant is project-scoped
  only), so the hooks never fired and the status bar, bell, and `prefix + g`
  jump silently stopped working. `tmux-claude-setup` now migrates any hooks left
  in that dead location back to `settings.json`.
- **The waiting bell reaches tmux again**, restoring the red window highlight.
  It was written to `/dev/tty`, which does not exist for hooks (they run without
  a controlling terminal), so the write failed: no bell, no highlight, and the
  failed redirect even leaked back as Stop-hook feedback. The BEL is now written
  to the waiting pane's real tty, resolved via tmux, and `tmux-claude-notify`
  always exits 0 so a failed bell can never surface as hook noise.

### Changed
- `tmux-claude-ensure-tmux` no longer requires `$TMUX` to be set. It configures
  any running tmux server directly (every option it sets is server-global) and
  is a quiet no-op when no server is running. Setup now takes effect immediately
  even when run from outside a tmux client.

## [0.1.2] - 2026-04-21

### Changed
- Hooks are now installed into `~/.claude/settings.local.json` instead of
  `~/.claude/settings.json`. `tmux-claude-setup` automatically migrates any
  tmux-claude hooks found in the legacy location, and `tmux-claude-teardown`
  scrubs both files.
- Status bar segment is now prepended to `status-right` instead of appended, so
  user-defined static content (e.g. `#H` for hostname) stays on the far right.
- Dropped the `.tmux-configured` marker fast-path in `tmux-claude-ensure-tmux`.
  Each hook fire now re-verifies the status-right segment, so reloading
  `.tmux.conf` re-integrates automatically instead of silently losing the
  Claude indicator until the tmux server restarts.
- `tmux-claude-teardown` now strips both the prepended and legacy appended
  forms of the status segment.

## [0.1.1] - 2026-04-09

### Changed
- `prefix + g` now clears the current pane's waiting state before jumping, so
  pressing it from inside a waiting pane advances to the next waiter instead of
  no-opping on the current one.

## [0.1.0] - 2026-04-08

### Added
- tmux status bar indicator showing when Claude Code is waiting for input
- `prefix + g` keybinding to jump to the oldest waiting Claude Code pane
- Bell notification when Claude Code enters waiting state
- Auto-clear of waiting status when switching to a pane
- One-time setup via `tmux-claude-setup` — works from anywhere, tmux
  configuration applied lazily on first hook fire
- Clean uninstall via `tmux-claude-teardown`
- Safe outside tmux — all hooks exit silently with no output
