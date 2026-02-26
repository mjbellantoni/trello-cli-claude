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

### `/create-parent-from-plan`

Set up a parent card with a numbered checklist of steps:

- "create a parent card for this plan"
- "make a parent Trello card with checklist"

**What it does:**
1. Creates a card with `[slug] title` format on Backlog
2. Adds plan/context as the description
3. Creates a "Steps" checklist with numbered items (01, 02, ...)

### `/start-next-child`

Start work on the next ready checklist item:

- "start next checklist item"
- "grab the next item and make a child card"

**What it does:**
1. Parses the parent's checklist for deps, migration tags, and completion status
2. Finds the first ready item (unchecked, deps satisfied, migration lock clear)
3. Creates a child card `[slug.NN] step title` on In Progress
4. Links the child back to the parent checklist item
5. Reports blocked items if nothing is ready

### `/complete-child`

Complete a child card and update the parent:

- "mark this child done"
- "complete this and check off parent"

**What it does:**
1. Moves child to Done/Committed or Done/Deployed (same logic as finish-card)
2. Checks off the corresponding parent checklist item

## Parent/Child Workflow

### Conventions

| Convention | Format | Example |
|-----------|--------|---------|
| Parent title | `[slug] description` | `[owner-data] Owner data ingestion` |
| Child title | `[slug.NN] step title` | `[owner-data.01] Create schema` |
| Checklist numbering | Two-digit, zero-padded | `01.` `02.` ... `10.` `11.` |
| Dependencies | `{deps:NN,NN}` | `{deps:01,02}` |
| Migration tag | `{migrations}` | `{migrations}` |

### Example Checklist

```
Steps: (1/4)
  [x] 01. Create owner schema {migrations}
  [ ] 02. Add API endpoints {deps:01}
  [ ] 03. Backfill job {deps:01,02}
  [ ] 04. Add ingestion worker {deps:03} {migrations}
```

Resulting child cards (created one at a time):
- `[owner-data.01] Create owner schema` — created first, now done
- `[owner-data.02] Add API endpoints` — ready (dep 01 is checked)
- `[owner-data.03] Backfill job` — blocked (dep 02 not done)
- `[owner-data.04] Add ingestion worker` — blocked (dep 03, and migration lock if 01 is still in Done/Committed)

### Migration Locking

Only one migration child may be in flight at a time. A migration child is "in flight" if it exists on **In Progress** or **Done/Committed** (not yet deployed). The lock clears when the card reaches **Done/Deployed**.

## License

MIT
