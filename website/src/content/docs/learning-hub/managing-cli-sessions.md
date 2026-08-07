---
title: 'Managing Sessions in the Copilot CLI'
description: 'Learn how to run multiple concurrent sessions, switch approval modes, undo agent changes, and start sessions in new worktrees from the Copilot CLI.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-07
estimatedReadingTime: '9 minutes'
tags:
  - copilot-cli
  - sessions
  - worktrees
  - fundamentals
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./github-copilot-app.md
  - ./automating-with-hooks.md
prerequisites:
  - GitHub Copilot CLI installed (v1.0.76+)
  - Basic familiarity with GitHub Copilot CLI sessions
---

As Copilot CLI has grown into a full agentic environment, session management has become a first-class feature. You can now run multiple sessions side-by-side in a split-view sidebar, control exactly how much autonomy the agent has at any moment, undo agent changes without needing git, and spin up fresh worktrees mid-session — all from within the CLI.

This article covers the key session management features added in Copilot CLI v1.0.76–1.0.78.

## Concurrent Sessions with the Sessions Sidebar

Starting with v1.0.76, Copilot CLI includes a **Sessions sidebar** that lets you manage multiple active sessions from a single terminal window. Switch between them, spawn new ones, and see each session's status at a glance — without opening additional terminal tabs.

### Enabling the Sessions Sidebar

The Sessions sidebar is available in experimental mode:

```
/experimental on
```

Once enabled, the sidebar appears on the right side of the CLI interface. You can:

- **Switch sessions** by selecting a session from the sidebar
- **Spawn a new session** directly from the sidebar
- **See status at a glance** — each session card shows what the agent is currently doing

> **Note**: In v1.0.79+, multiple concurrent sessions are available without requiring experimental mode. Sessions are also accessible from a dedicated **Sessions tab** in the navigation.

### Why Run Multiple Sessions?

Running concurrent sessions lets you parallelize work the same way you would with multiple terminal tabs — but with better context management:

| Scenario | Single session | Multiple sessions |
|----------|---------------|-------------------|
| Long-running refactor | Blocks you from other tasks | Run in background while you work elsewhere |
| Research + implementation | Context gets mixed | Each session stays focused |
| Two independent features | Context switching | Each agent tracks its own task independently |
| Review while building | Must stop/start | Sessions run in parallel |

Each session maintains its own conversation history, working directory, and tool state. Switching sessions is instant — no restarting MCP servers or rebuilding hook state.

### Keeping Prompts with Their Sessions

When you switch between sessions, any text you've typed in the prompt box stays with the session it was typed for (since v1.0.76). Your draft prompt won't follow you when you switch away.

## Switching Approval Modes with `/permissions`

*(Available since v1.0.78)*

The `/permissions` command lets you change the agent's approval mode at any point during a session:

```
/permissions
```

This opens a picker to switch between:

| Mode | Behavior |
|------|----------|
| **Interactive** | The agent asks for approval before significant actions (default) |
| **Autopilot** | The agent proceeds autonomously without confirmation prompts |
| **Plan** | The agent proposes a plan and waits for approval before executing |

### Switching Modes Mid-Session

You can switch modes at any time — you don't need to restart the session. For example:

1. Start in **Plan** mode to review what the agent intends to do
2. Switch to **Autopilot** once you've approved the approach
3. Switch back to **Interactive** when the agent reaches a decision point that needs your judgment

### Autopilot Persistence

In v1.0.76+, autopilot mode **stays selected after task completion** by default. When the agent finishes a task in autopilot, it remains in autopilot for the next task instead of reverting to interactive. To revert to interactive after each task, set `stayInAutopilot` to `false` in your settings.

### Pre-v1.0.78: Setting Mode at Startup

Before `/permissions` was available, you could only set the approval mode at CLI startup:

```bash
copilot --autopilot    # start in autopilot
copilot --plan         # start in plan mode
```

The `/permissions` command makes it possible to change modes dynamically without restarting.

## Undoing Agent Changes with `/rewind`

*(Significantly improved in v1.0.78)*

The `/rewind` command undoes changes the agent made to your files. In v1.0.78, `/rewind` was substantially improved:

- **No longer requires git** — works in any directory, not just git repositories
- **Surgical rollback** — restores only the files Copilot changed, skipping any file whose contents no longer match what Copilot last wrote
- **Two rollback options** — choose between undoing just the conversation or undoing both the conversation and the file changes

### Using `/rewind`

```
/rewind
```

You'll be prompted to choose:

| Option | What it does |
|--------|-------------|
| **Conversation only** | Rolls back the conversation turn without touching any files |
| **Conversation + files** | Rolls back the conversation and restores all files the agent modified in that turn |

### When to Use `/rewind`

- The agent made unexpected changes and you want to start over from a known state
- You approved an action and realized it wasn't what you wanted
- The agent's output was correct but you want to explore a different approach
- You're in a repository without git (or in a directory where git isn't the right undo mechanism)

> **Tip**: `/rewind` only restores files that still match what Copilot last wrote. If you made additional edits to a file after the agent changed it, those files are left untouched — your edits are preserved.

## Starting Sessions in New Worktrees

*(Available as `/new-worktree` experimental in v1.0.78, renamed to `/worktree new` in v1.0.79)*

The `/worktree new` command (or `/new-worktree` in v1.0.78) creates a new git worktree and starts a fresh conversation in it:

```
/worktree new
```

Or using the experimental command in v1.0.78:

```
/new-worktree
```

### Why Start a Session in a New Worktree?

This is useful when you're mid-session and realize you want to explore a completely different approach without disturbing your current work:

1. Your current session has changes you want to preserve
2. You want a clean branch to try an alternative implementation
3. You want to hand off a subtask to a fresh agent session with isolated context

Each worktree is a real, isolated copy of your repository on a separate branch. The new session starts in the new worktree with a clean conversation history. Your original session continues exactly where it was.

### Worktrees and the Copilot App

If you're using the **GitHub Copilot app**, worktree isolation is handled automatically — every session the app creates runs in its own worktree. The `/worktree new` command brings the same pattern to the CLI, letting you manage worktree-based sessions directly from the terminal.

See [Getting Started with the GitHub Copilot app](../github-copilot-app/) for more on the app's parallel session model.

## Monitoring Resource Usage with `/limits predict`

*(Available since v1.0.76)*

To estimate how much of your AI credit budget a session will use before it finishes, use:

```
/limits predict
```

This analyzes your current session against similar past sessions and suggests an appropriate credit limit. Useful for:

- Long-running autonomous tasks where you want to set a budget
- Understanding whether your current plan fits within your monthly quota
- Avoiding unexpected credit exhaustion on complex tasks

## Quick Reference

| Command | What it does | Available since |
|---------|-------------|-----------------|
| `/experimental on` | Enable experimental features (including Sessions sidebar) | v1.0.76 |
| `/permissions` | Switch between interactive / autopilot / plan modes | v1.0.78 |
| `/rewind` | Undo agent changes (conversation-only or conversation+files) | Improved v1.0.78 |
| `/new-worktree` | Create worktree + start new session (experimental) | v1.0.78 |
| `/worktree new` | Create worktree + start new session | v1.0.79+ |
| `/limits predict` | Estimate AI credit usage for this session | v1.0.76 |

## Further Reading

- **Changelog**: [github/copilot-cli releases](https://github.com/github/copilot-cli/releases) — full release notes for each version
- **Worktrees in the Copilot app**: [Getting Started with the GitHub Copilot app](../github-copilot-app/) — parallel sessions via the desktop app
- **Coding agent**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — autonomous coding with hooks and remote control
- **Automating with Hooks**: [Automating with Hooks](../automating-with-hooks/) — deterministic guardrails for session lifecycle events

---
