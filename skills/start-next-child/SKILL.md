---
name: start-next-child
description: Use when user says "start next checklist item", "grab the next item and make a child card", "start next child", or wants to begin work on the next step of a parent card
---

# Start Next Child

## Overview

Find the next ready checklist item on a parent card, create a child card for it, move it to In Progress, and link it back to the parent checklist. Respects dependency ordering and migration locking.

## Conventions

- **Parent title format:** `[<slug>] <title>`
- **Child title format:** `[<slug>.<NN>] <step title>`
- **Deps:** `{deps:01,02}` — item blocked until those checklist items are checked off
- **Migrations:** `{migrations}` — only one migration child in flight at a time
- **Child link:** Appended to checklist item as `→ <child-url>`

## The Process

### Phase 1: Fetch Parent Card

```bash
bin/trello card show <parent-ref>
```

Extract slug from title — must match `/^\[(?<slug>[a-z0-9-]+)\]/`.
If no slug found, STOP and tell the user to add a `[slug]` prefix to the card title.

### Phase 2: Load and Parse Checklist

Find the "Steps" checklist in the card show output. If no checklist named "Steps", use the first (and only) checklist. If multiple checklists and none named "Steps", STOP and ask.

Parse each checklist item into these fields:

| Field | How to extract |
|-------|---------------|
| number | Leading `NN.` (two+ digits before first period) |
| title | Text after `NN. ` and before any `{...}` tags or `→` link |
| deps | Numbers from `{deps:NN,NN}` tag (accept spaces around commas) |
| migrations | `true` if `{migrations}` tag present (case-insensitive) |
| checked | `[x]` prefix in card show output |
| has_child | `true` if item contains `trello.com/c/` URL or `→` |

### Phase 3: Check Migration Lock

**Only needed if any unchecked item has `{migrations}`.**

Scan for open migration children across these lists:

```bash
bin/trello list cards "In Progress"
bin/trello list cards "Done/Committed"
```

For each card in the output whose title matches `/^\[[a-z0-9-]+\.\d{2,}\]/` (child card pattern):

```bash
bin/trello card show <card-ref>
```

Check the description for `requires_migrations: true`. If found, migrations are **locked** — note which card holds the lock.

**Optimization:** Skip the `card show` calls if no unchecked item has `{migrations}`. Also, only inspect cards matching the child title pattern.

### Phase 4: Determine Ready Items

An item is READY if ALL of:
1. Not checked
2. No existing child link (no `→` or trello URL in the item text)
3. All deps are checked off (every number in its `{deps:...}` list corresponds to a checked item)
4. If `{migrations}` is true: migration lock is clear (Phase 3 found no open migration child)

### Phase 5: Pick Next or Report Blocked

**If a ready item exists:** pick the one with the smallest NN.

**If NO ready items exist:** output a blocked report and STOP:

```
No ready items. Blocked:
- 03 blocked: deps 01, 02 not done
- 04 blocked: migrations locked by [other-slug.02] (In Progress)
- 05 blocked: deps 03 not done
```

Keep it short — one line per blocked item.

### Phase 6: Create Child Card

Build the child card description with metadata block:

```
---
parent_card: <parent card URL>
parent_slug: <slug>
parent_checklist_name: Steps
parent_item_number: <NN>
requires_migrations: <true|false>
deps: <comma-separated list or empty>
---

Goal: <step title from checklist item>
Notes: parent card has full context; refer there.
```

Create the card:

```bash
bin/trello card new "[<slug>.<NN>] <step title>" \
  --description "<metadata block + goal>" \
  --list "In Progress"
```

No need to `card move` separately — `--list "In Progress"` creates it there directly.

### Phase 7: Update Parent Checklist Item

Append the child card URL to the checklist item text:

```bash
bin/trello checklist item-edit <parent-ref> "Steps" "<exact current item text>" "<current item text> → <child-url>"
```

**Important:** Use the exact current item text as the ITEM argument (not a position number), since `item-edit` matches by exact name. The NEW_TEXT is the full replacement including the appended link.

### Phase 8: Confirm

Output:
- Child card URL
- One sentence: "Working on: [slug.NN] <step title>"
- If any items were skipped as blocked, add a short note

Example:
```
Created: https://trello.com/c/xyz789
Working on: [owner-data.03] Backfill job
(Skipped 02: deps 01 not done)
```

## Red Flags

If you catch yourself doing these, STOP:

- **Creating multiple child cards** — Only ONE child per invocation
- **Creating a child for a checked item** — Checked means done
- **Creating a duplicate child** — If item already has `→ <url>`, skip it
- **Ignoring deps** — Always verify dep items are checked
- **Ignoring migration lock** — Always check if `{migrations}` items need the lock scan
- **Guessing list names** — Use exact names: "In Progress", "Done/Committed"
- **Using position numbers for item-edit** — Use exact item text as the ITEM argument
- **Skipping the metadata block** — Every child MUST have the `---` metadata block in its description

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Fetch | `bin/trello card show` | Get parent details |
| 2. Parse | (text parsing) | Extract items, deps, tags |
| 3. Migration | `bin/trello list cards` + `card show` | Check lock |
| 4. Ready | (logic) | Filter to actionable items |
| 5. Pick | (logic) | Smallest NN or blocked report |
| 6. Create | `bin/trello card new --list "In Progress"` | Child card |
| 7. Link | `bin/trello checklist item-edit` | Append `→ url` |
| 8. Confirm | (output) | URL + one sentence |
