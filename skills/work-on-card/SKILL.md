---
name: work-on-card
description: Use when user says "fix #123", "work on card xyz", "tackle that bug", "let's debug card abc", or references a Trello card to start working on
---

# Work on Card

## Overview

Fetch a Trello card, download its attachments, move it to In Progress, and trigger the appropriate workflow based on card type.

## The Process

### Phase 1: Parse Reference

Recognize card references in natural language:
- "#123" or "card 123" → card number
- "abc123XY" → short link
- "https://trello.com/c/abc123XY" → extract short link
- "that card" / "the bug" → ask for reference

### Phase 2: Fetch Card

```bash
bin/trello card show <ref>
```

Read and understand:
- **Title** - The problem/feature summary
- **Description** - Full context, steps, expected behavior
- **Labels** - Determines which skill to trigger
- **Checklists** - Existing progress or subtasks
- **Attachments** - Screenshots, stack traces, files

### Phase 3: Download Attachments

```bash
bin/trello attach list <ref>
```

For each attachment:
```bash
bin/trello attach get <ref> <filename>
```

Then:
- **Screenshots** → View with Read tool (handles images)
- **Stack traces** → Read content
- **Other files** → Note location for reference

### Phase 4: Move to In Progress

```bash
bin/trello card move <ref> "In Progress"
```

### Phase 5: Trigger Downstream Skill

**Check labels and route:**

| Label | Action |
|-------|--------|
| bug | Invoke `superpowers:systematic-debugging` |
| (any other or none) | Invoke `superpowers:brainstorming` |

**Hand off context:**
- Card title = problem/feature description
- Card description = full context
- Downloaded attachments = evidence/reference
- Checklists = existing analysis or tasks

## Red Flags

If you catch yourself doing these, STOP:

- **Starting work without moving to In Progress** - Always move first
- **Skipping attachments** - They contain key context
- **Guessing the skill** - Check the label
- **Summarizing for downstream** - Let the skill read the full context
- **Asking "what should I do?"** - The card tells you

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Parse | (natural language) | Extract card reference |
| 2. Fetch | `bin/trello card show` | Get card details |
| 3. Download | `bin/trello attach get` | Get all attachments |
| 4. Move | `bin/trello card move` | Mark as In Progress |
| 5. Route | Check labels | Trigger debugging or brainstorming |
