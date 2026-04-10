# Claude Squad Fork

## Project Profile
- **purpose**: Fork of claude-squad (smtg-ai/claude-squad) with enhancements for multi-account, multi-repo, and power-user workflows
- **owner**: Franz Felberer
- **context**: Personal project — replacing/augmenting Conductor for parallel AI agent management
- **tech**: Go (1.23+), Bubble Tea TUI framework, tmux, git worktrees, Cobra CLI
- **upstream**: https://github.com/smtg-ai/claude-squad (AGPL-3.0, v1.0.17)
- **current_focus**: Initial fork setup and priority enhancements
- **keywords**: claude-squad, multi-agent, tmux, worktree, parallel agents, multi-account, fork
- **workflow_mode**: gsd-full

## Upstream Architecture

Claude Squad is a Go TUI that manages parallel AI coding agent sessions using three layers:

1. **tmux** — terminal isolation. Each session runs in `claudesquad_<title>_<timestamp>`
2. **git worktrees** — filesystem isolation. Each session gets `~/.claude-squad/worktrees/<path>/` on a dedicated branch
3. **Bubble Tea TUI** — Go-based terminal UI with Lipgloss styling, Cobra CLI

### Key source structure
```
app/          - Main state machine, home model (GlobalInstanceLimit lives here)
cmd/          - Cobra CLI commands (root, reset, debug, version)
config/       - Configuration management, profiles
daemon/       - Background auto-yes daemon
keys/         - Keyboard bindings
session/      - Instance lifecycle, tmux integration, git worktree subsystem
ui/           - TUI components
main.go       - Entry point
```

### Current config format (~/.claude-squad/config.json)
```json
{
  "default_program": "claude",
  "auto_yes": false,
  "daemon_poll_interval": 1000,
  "branch_prefix": "<username>/",
  "profiles": [
    {"name": "claude", "program": "claude"},
    {"name": "aider-gpt4", "program": "aider --model gpt-4"}
  ]
}
```

## Enhancement Plan (Priority Order)

### 1. Raise session limit (5 min)
- Change `GlobalInstanceLimit = 10` in `app/app.go` to 25 or make configurable
- Add `max_instances` to config.json

### 2. Per-profile environment variables (few hours)
- Add `env` map to profile config: `{"env": {"CLAUDE_CONFIG_DIR": "~/.claude-personal"}}`
- Inject env vars into tmux session creation
- **Key insight**: `CLAUDE_CONFIG_DIR` lets each profile have its own OAuth session, so different profiles can use different Max subscriptions without API keys
- Example config:
```json
{
  "profiles": [
    {
      "name": "personal",
      "program": "claude",
      "env": {"CLAUDE_CONFIG_DIR": "~/.claude-personal"}
    },
    {
      "name": "geminicap",
      "program": "claude",
      "env": {"CLAUDE_CONFIG_DIR": "~/.claude-geminicap"}
    }
  ]
}
```

### 3. CLI session management (1-2 days)
- Add non-interactive commands: `cs new --name "task" --prompt "do X" --profile personal`
- Add `cs list`, `cs pause <name>`, `cs resume <name>`, `cs kill <name>`
- Internal `createInstance()` already exists — needs non-interactive code path

### 4. Multi-repo support (1-2 days)
- Allow sessions to specify `repo_path` instead of requiring `cs` to run from within a git repo
- Store repo association per instance in JSON state file
- Show repo name in TUI session list

### 5. Per-profile branch prefix (30 min)
- Already has global `branch_prefix` — make it per-profile

## Known Upstream Issues to Be Aware Of

- **macOS TUI bug** (#271): Invisible input field on macOS with various terminals
- **Auto-yes race condition** (#266): Prompts sent at session creation can be dropped
- **No merge workflow**: Manual merge only after checkout, no in-TUI merge
- **Worktree setup hook**: PRs #268/#270 in progress upstream — auto-install deps after worktree creation
- **CLAUDE_SQUAD_HOME**: PR #246 not merged — custom config dir location

## User Context

Franz runs 13+ parallel Conductor workspaces across multiple repos:
- Personal projects: `/Users/work/Personal/Source/`
- Geminicap (business): `/Users/work/Source/geminicap/`
- Quartz (client): `/Users/work/Source/Quartz/`

Two Anthropic accounts:
- Personal: franz@felberer.at (Max plan)
- Business: franz@geminicap.co (Team plan)

Heavy use of Claude Code features that will carry over: CLAUDE.md, hooks, MCP servers, statusline.
The multi-account separation via CLAUDE_CONFIG_DIR per profile is the most valuable enhancement — it solves a problem Conductor itself cannot solve.

## GitHub
- Account: FranzFelberer (personal)
- Upstream remote: `upstream` -> smtg-ai/claude-squad
- Origin remote: `origin` -> FranzFelberer/claude-squad-fork
