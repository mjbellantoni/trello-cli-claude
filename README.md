# trello-cli-claude

A [Claude Code](https://claude.ai/claude-code) plugin for natural language Trello integration. Manage your Trello cards, track bugs, and organize tasks directly from your Claude Code sessions.

## Overview

This plugin wraps [trello-cli](https://github.com/mjbellantoni/trello-cli), providing Claude Code with skills to:

- **Work on cards** - Fetch card details, download attachments, and start work
- **File bugs** - Capture session context into new Trello cards
- **Finish cards** - Move completed cards to the appropriate Done list

## Prerequisites

1. **trello-cli** must be installed and configured in your project:
   - Add `trello-cli` to your Gemfile
   - Run `bundle install && bundle binstubs trello-cli`
   - Create `.trello.yml` with your board ID and default list
   - Set `TRELLO_API_KEY` and `TRELLO_TOKEN` environment variables

See the [trello-cli documentation](https://github.com/mjbellantoni/trello-cli) for setup details.

## Installation

### Via Plugin Marketplace

```shell
/plugin marketplace add mjbellantoni/trello-cli-claude
/plugin install trello-cli-claude
```

### Local Development

```shell
claude --plugin-dir /path/to/trello-cli-claude
```

## Skills

### `/work-on-card`

Start working on a Trello card. Recognizes natural language references:

- "fix #123"
- "work on card abc123"
- "tackle that bug"

**What it does:**
1. Fetches card details (title, description, labels, checklists)
2. Downloads all attachments (screenshots, stack traces)
3. Moves card to "In Progress"
4. Routes to appropriate workflow based on labels (bug → debugging, other → brainstorming)

### `/file-bug`

Capture an issue you encountered into a new Trello card:

- "file a bug for that"
- "create a ticket"
- "that's a bug - capture it"

**What it does:**
1. Gathers context from the session (error, steps, expected/actual behavior)
2. Formats a structured bug report
3. Creates card with `bug` label at top of Inbox
4. Attaches stack traces or screenshots if available

### `/finish-card`

Complete a card and move it to the appropriate Done list:

**What it does:**
1. Checks for recent commits to determine destination
2. Moves to "Done/Committed" (if code was merged) or "Done/Deployed" (if no code changes)
3. Asks for clarification if ambiguous

## License

MIT
