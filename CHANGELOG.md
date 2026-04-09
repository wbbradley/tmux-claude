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
