---
name: create-parent-from-plan
description: Use when user says "create a parent card for this plan", "make a parent Trello card with checklist", or wants to set up a parent card with numbered steps
---

# Create Parent from Plan

## Overview

Create a parent Trello card with a numbered checklist of steps. The parent stays in Backlog permanently. Each checklist item becomes a child card later via start-next-child.

## Conventions

- **Parent title:** `[<slug>] <title>` — slug is lowercase alphanumeric + hyphens
- **Checklist items:** Prefixed with slug and number: `[slug.01]` `[slug.02]` `[slug.03]` etc.
- **Deps syntax:** Append `{deps:01,02}` to items that depend on earlier steps
- **Migrations syntax:** Append `{migrations}` to items that involve Rails migrations

## The Process

### Phase 1: Gather Inputs

Collect from the user (ask if not provided):
1. **slug** — short identifier, e.g. `owner-data` (must match `[a-z0-9-]+`)
2. **title** — human-readable title
3. **plan/context text** — description content (verbatim or lightly formatted)
4. **steps** — ordered list of step titles, with optional `{deps:...}` and `{migrations}` tags
5. **list name** — default: `Backlog`

### Phase 2: Create the Card

```bash
bin/trello card new "[<slug>] <title>" \
  --description "<plan/context text>" \
  --list "<list name>"
```

Capture the card URL from output (line starting with "Created:").

### Phase 3: Add the Steps Checklist

```bash
bin/trello checklist add <card-ref> "Steps"
```

For each step, add a numbered checklist item:

```bash
bin/trello checklist item-add <card-ref> "Steps" "[<slug>.01] <step title>"
bin/trello checklist item-add <card-ref> "Steps" "[<slug>.02] <step title> {deps:01}"
bin/trello checklist item-add <card-ref> "Steps" "[<slug>.03] <step title> {deps:01,02} {migrations}"
```

Number format: always two digits, zero-padded (01, 02, ... 09, 10, 11, ...).

### Phase 4: Confirm

Report ONLY: the card URL.

Example: `Created [owner-data] parent: https://trello.com/c/abc123XY`

## Red Flags

If you catch yourself doing these, STOP:

- **Creating child cards** — This skill only creates the parent. Children are created lazily by start-next-child.
- **Forgetting the slug brackets** — Title MUST be `[slug] title`, not just `slug title`
- **Using single-digit numbers** — Always zero-pad: `[slug.01]` not `[slug.1]`
- **Adding items without the bracket prefix** — Every item starts with `[<slug>.NN]`
- **Verbose confirmation** — Just the URL, nothing more

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Gather | (conversation) | Get slug, title, plan, steps |
| 2. Create | `bin/trello card new` | Parent card on Backlog |
| 3. Steps | `bin/trello checklist add` + `item-add` | Numbered checklist |
| 4. Confirm | (output) | Card URL only |
