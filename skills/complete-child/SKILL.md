---
name: complete-child
description: Use when user says "mark this child done", "complete this and check off parent", "finish this child card", or wants to complete a child card and update the parent checklist
---

# Complete Child

## Overview

Move a child card to the appropriate Done list, then check off the corresponding item on the parent card's checklist. Reuses the same Done list logic as finish-card.

## The Process

### Phase 1: Fetch Child Card

```bash
bin/trello card show <child-ref>
```

Find the `### 🔗 Lineage` section in the description and parse the bullet list:

- **Parent URL:** extract from the Markdown link in the `**Parent:**` bullet — match `[text](url)` and take the `url`
- **Parent checklist name:** text after `**Checklist:**` up to the `·` separator, trimmed
- **Parent item number:** text after `**Item:**` on the same line, trimmed
- **Migrations:** text after `**Migrations:**` up to the `·` separator, trimmed — `yes` or `no`

**Fallback for older cards (YAML frontmatter):** If no `### 🔗 Lineage` section is found, look for a metadata block between `---` delimiters and parse `parent_card`, `parent_slug`, `parent_checklist_name`, `parent_item_number` as key-value pairs.

**Last-resort fallback:** Extract `[slug.NN]` from the child card title using `/^\[(?<slug>[a-z0-9-]+)\.(?<nn>\d{2,})\]/`. Search the description for a trello.com URL as the parent reference. Use "Steps" as default checklist name.

### Phase 2: Move Child to Done List

Reuse finish-card logic:

**Check for recent commits:**

```bash
git log --oneline --since="2 hours ago"
```

**Decision:**
- If commits relate to this work → `Done/Committed`
- If `/commit` skill was used in this session → `Done/Committed`
- If pure research/planning → `Done/Deployed`
- If unclear → ask user:

```
question: "Which Done list should this child move to?"
header: "Destination"
options:
  - label: "Done/Committed"
    description: "Code was committed/merged — PR open or merged, not yet deployed"
  - label: "Done/Deployed"
    description: "No code changes, or already deployed to production"
```

```bash
bin/trello card move <child-ref> "<chosen Done list>"
```

### Phase 3: Check Off Parent Item

First, fetch the parent card to inspect checklist state:

```bash
bin/trello card show <parent-ref>
```

If the parent card is not found, STOP and tell the user the parent reference from the child's metadata is invalid.

If the checklist (`<checklist-name>`) does not exist on the parent, STOP and tell the user.

**Idempotency:** If the item is already checked (visible in `card show` output as `[x]`), skip the check command and note it in the confirmation.

If not already checked, check it off:

```bash
bin/trello checklist item-check <parent-ref> "<checklist-name>" "<NN>"
```

Where `<NN>` is the 1-based position from `parent_item_number`. Since checklist items are numbered 01, 02, etc., item number 01 is position 1, item 02 is position 2, and so on.

### Phase 4: Confirm

Report ONLY: `Checked off <NN> on [<slug>] and moved child to <Done list>.`

If already checked: `Item <NN> on [<slug>] was already checked. Moved child to <Done list>.`

## Red Flags

If you catch yourself doing these, STOP:

- **Skipping the parent update** — Always check off the parent item
- **Moving to wrong Done list** — Check git history, ask if ambiguous
- **Double-checking an already-checked item** — Check `card show` first
- **Guessing the parent** — Parse metadata block or title, don't guess
- **Verbose confirmation** — One sentence only

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Fetch | `bin/trello card show` | Get child details + metadata |
| 2. Move | `bin/trello card move` | Child → Done list |
| 3. Check | `bin/trello card show` + `checklist item-check` | Verify parent, mark item done |
| 4. Confirm | (output) | One sentence |
