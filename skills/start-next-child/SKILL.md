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
- **Step type:** `{research}`, `{prototype}` or `{grilling}` — the step's output is an answer, not shipped code. Untagged means implementation.
- **Child link:** Description text wrapped as Markdown link: `[description](child-url)`

## The Process

### Phase 1: Fetch Parent Card

```bash
bin/trello card show <parent-ref>
```

Extract slug from title — must match `/^\[(?<slug>[a-z0-9-]+)\]/`.
If no slug found, STOP and tell the user to add a `[slug]` prefix to the card title.

Extract labels from the `Labels:` line in the card show output (case-insensitive match). Save these for Phase 6:

- **Kind:** the one label out of `bug`, `feature`, `chore`. If the parent has none of them, or more than one, STOP and ask which kind the children take.
- **Origin:** `user`, if present.

### Phase 2: Load and Parse Checklist

Find the "Steps" checklist in the card show output. If no checklist named "Steps", use the first (and only) checklist. If multiple checklists and none named "Steps", STOP and ask.

Parse each checklist item into these fields:

| Field | How to extract |
|-------|---------------|
| number | New format: inside `[slug.NN]` bracket prefix. Old format: leading `NN.` before first space. |
| title | Text between the prefix (`[slug.NN] ` or `NN. `) and any `{...}` tags. If the title is a Markdown link `[text](url)`, extract the text portion. Trim trailing whitespace. |
| deps | Numbers from `{deps:NN,NN}` tag (accept spaces around commas) |
| migrations | `true` if `{migrations}` tag present (case-insensitive) |
| step_type | `research`, `prototype` or `grilling` from a bare tag of that name (case-insensitive). `implementation` when no such tag is present. |
| checked | `[x]` prefix in card show output |
| has_child | `true` if item contains `trello.com/c/` URL |

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
2. No existing child link (no trello URL in the item text)
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

Create the child with the command for the parent's kind: `bin/trello chore new`, `feature new` or `bug new`. These commands require the kind's sections and enforce its word cap. Never use `card new`, which skips both.

Fill the sections from the step's entry in the parent's plan. The plan is the parent's description or its attached markdown design (`bin/trello attach list <parent-ref>`, then `bin/trello attach get`). Keep each section to this one step. The parent carries the full context.

| Kind | Section | Content |
|------|---------|---------|
| `chore` | `--what` | The step's work, in one or two sentences |
| `chore` | `--why-now` | "Step <NN> of [<slug>]." plus what the step unblocks |
| `chore` | `--done-when` | One observable condition per value |
| `feature` | `--what` | The step's capability, in one or two sentences |
| `feature` | `--why` | "Step <NN> of [<slug>]." plus who wants it |
| `feature` | `--done-when` | One Given/When/Then per value |
| `bug` | `--steps` | The parent's steps to recreate, narrowed to this step |
| `bug` | `--expected` | What this step makes true |
| `bug` | `--actual` | What happens before this step |

The lineage goes in `--notes`, as this bullet list with no heading:

```
- **Parent:** [<parent card title>](<parent card URL>)
- **Checklist:** <checklist name> · **Item:** <NN>
- **Migrations:** <yes|no> · **Deps:** <comma-separated list or "none">
- **Step type:** <research|prototype|grilling|implementation>
```

The parent link MUST be a Markdown link `[title](url)` — do NOT use a naked URL.

```bash
bin/trello chore new "[<slug>.<NN>] <step title>" \
  --what "<what>" \
  --why-now "<why now>" \
  --done-when "<condition>" "<condition>" \
  --notes "<lineage bullets>" \
  --list "In Progress"
```

Add `--label user` when the parent carries `user`.

If the command rejects the input as over the word cap, do not reword to fit. The step is too big for one card. STOP and tell the user.

No need to `card move` separately — `--list "In Progress"` creates it there directly.

### Phase 7: Update Parent Checklist Item

Wrap the description text in a Markdown link to the child card. The `[slug.NN]` prefix and any `{...}` tags stay unchanged — only the description becomes a link.

**Before:** `[slug.02] Step description {deps:01}`
**After:**  `[slug.02] [Step description](https://trello.com/c/xyz789) {deps:01}`

```bash
bin/trello checklist item-edit <parent-ref> "Steps" "<exact current item text>" "<text with description wrapped as link>"
```

**Important:** Use the exact current item text as the ITEM argument (not a position number), since `item-edit` matches by exact name.

**Old format items:** If the item uses old `NN. title` format, convert it to the new format at the same time: `[slug.NN] [title](url) {tags}`.

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
- **Creating a duplicate child** — If item already has a Markdown link or trello URL, skip it
- **Ignoring deps** — Always verify dep items are checked
- **Ignoring migration lock** — Always check if `{migrations}` items need the lock scan
- **Guessing list names** — Use exact names: "In Progress", "Done/Committed"
- **Using position numbers for item-edit** — Use exact item text as the ITEM argument
- **Appending URL instead of linking** — Wrap the description in `[text](url)`, do NOT append `→ url` or bare URLs
- **Using `card new`** — Create the child with `bin/trello <kind> new`, where the kind comes from the parent's label
- **Skipping the lineage** — Every child MUST carry the lineage bullets in its Notes
- **Using a naked URL for parent** — MUST use Markdown link `[title](url)` format

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Fetch | `bin/trello card show` | Get parent details, kind and origin labels |
| 2. Parse | (text parsing) | Extract items, deps, tags |
| 3. Migration | `bin/trello list cards` + `card show` | Check lock |
| 4. Ready | (logic) | Filter to actionable items |
| 5. Pick | (logic) | Smallest NN or blocked report |
| 6. Create | `bin/trello <kind> new --list "In Progress"` | Child card, lineage in Notes |
| 7. Link | `bin/trello checklist item-edit` | Wrap description as Markdown link |
| 8. Confirm | (output) | URL + one sentence |
