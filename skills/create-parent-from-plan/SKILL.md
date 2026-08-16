---
name: create-parent-from-plan
description: Use when user says "create a parent card for this plan", "make a parent Trello card with checklist", or wants to set up a parent card with numbered steps
---

# Create Parent from Plan

## Overview

Create a parent Trello card with a numbered checklist of steps, in the list the caller names. Each checklist item becomes a child card later, one at a time, via start-next-child.

**Why the checklist and not a pile of cards:** the checklist is the commitment; the child cards are created lazily. Do not create cards speculatively and do not do design speculatively. A step you have not started yet is a line of text you can rewrite for free — a card you created in advance is a card someone has to read, triage, and eventually close. Write down the whole shape of the work, then build it out one step at a time.

## Conventions

- **Parent title:** `[<slug>] <title>` — slug is lowercase alphanumeric + hyphens
- **Checklist items:** Prefixed with slug and number: `[slug.01]` `[slug.02]` `[slug.03]` etc.
- **Deps syntax:** Append `{deps:01,02}` to items that depend on earlier steps
- **Migrations syntax:** Append `{migrations}` to items that involve Rails migrations
- **Dated lists:** Named `By Mmm DD` — e.g. `By Aug 30`, `By Sep 12`. These are
  deadlines, not sprint starts, so a dated list is normally in the future. This
  is context for whoever picks the list, not a rule for this skill to apply.

## Parent Lifecycle

The parent is not a permanent parking spot — it moves with the work.

| Stage | List | Trigger |
|-------|------|---------|
| Created | the list the caller named | This skill. Carries the Steps checklist and usually an attached markdown high-level design. |
| Active | `In Progress` | The first child card is created. From here the parent follows the normal flow. |
| Finished | `Done/Deployed` | Every checklist item is checked and every child has reached Done/Deployed. |

The moves after creation are not this skill's job — they happen as children are
started and completed:

```bash
bin/trello card move <parent-ref> "In Progress"
bin/trello card move <parent-ref> "Done/Deployed"
```

There is no `Backlog` list on this board. Never use it.

## The Process

### Phase 1: Gather Inputs

Collect from the user (ask if not provided):
1. **slug** — short identifier, e.g. `owner-data` (must match `[a-z0-9-]+`)
2. **title** — human-readable title
3. **plan/context text** — description content (verbatim or lightly formatted)
4. **steps** — ordered list of step titles, with optional `{deps:...}` and `{migrations}` tags
5. **high-level design** — a markdown file to attach, if one exists. Most parents have one.
6. **list** — the list the parent goes in. Take it from the request if the
   caller named one; otherwise Phase 2 asks.

### Phase 2: Confirm the Target List

If the caller already named a list, use it. You are done — skip to Phase 3.

Otherwise, show them the board and ask:

```bash
bin/trello list all
```

Ask which list the parent goes in. Use the answer verbatim.

**Do not infer the list.** Not from today's date, not from which list looks
newest, not from a name that happens to parse as a date. The caller knows where
the work belongs and you do not; a wrong guess files the parent somewhere nobody
will look for it. Asking costs one question.

Do not create a list and do not invent a name. If the caller names a list that
does not exist, `card new` in Phase 3 will fail — report that and ask again
rather than substituting a list you think they meant.

### Phase 3: Create the Card

```bash
bin/trello card new "[<slug>] <title>" \
  --description "<plan/context text>" \
  --list "<list from Phase 2>"
```

Capture the card URL from output (line starting with "Created:").

### Phase 4: Attach the High-Level Design

If a design document exists:

```bash
bin/trello attach upload <card-ref> <path-to-design.md>
```

The design comes from the design conversation upstream — brainstorming, planning,
whatever produced it — and normally lives in `doc/scratchpad/` or `docs/plans/`.
Attach that file as it is.

Skip only when there is genuinely no design document. Don't write one to satisfy
this phase — an attached design is a document that already existed because the
work needed it.

### Phase 5: Add the Steps Checklist

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

### Phase 6: Confirm

Report ONLY: the card URL and the list it landed in.

Example: `Created [owner-data] parent in By Aug 30: https://trello.com/c/abc123XY`

## Red Flags

If you catch yourself doing these, STOP:

- **Using `Backlog`** — That list does not exist on this board.
- **Creating child cards** — This skill only creates the parent. Children are created lazily by start-next-child.
- **Designing steps you haven't committed to** — Steps are titles, not specs. The design work happens when the child card is started.
- **Picking the list yourself** — Ask. Date arithmetic on list names is how this skill filed work into the wrong place; there is no rule to apply and no date to parse.
- **Creating a list** — Ask the user. Don't invent board structure.
- **Forgetting the slug brackets** — Title MUST be `[slug] title`, not just `slug title`
- **Using single-digit numbers** — Always zero-pad: `[slug.01]` not `[slug.1]`
- **Adding items without the bracket prefix** — Every item starts with `[<slug>.NN]`
- **Moving the parent yourself** — It moves to In Progress when the first child is created, not now
- **Verbose confirmation** — Just the URL and list, nothing more

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Gather | (conversation) | Get slug, title, plan, steps, design doc, list |
| 2. Confirm list | `bin/trello list all` | Only if the caller did not name one — then ask |
| 3. Create | `bin/trello card new --list "<list>"` | Parent card |
| 4. Attach | `bin/trello attach upload` | High-level design markdown |
| 5. Steps | `bin/trello checklist add` + `item-add` | Numbered checklist |
| 6. Confirm | (output) | Card URL + list |
