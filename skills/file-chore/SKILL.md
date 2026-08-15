---
name: file-chore
description: Use when user says "file a chore", "capture this cleanup", "that's tech debt - make a card", or wants to record maintenance work with no user-facing behavior change
---

# File Chore

## Overview

Capture maintenance work — cleanup, upgrades, tech debt, tooling — into a Trello card so work can continue later. Returns just the card reference - keeps context minimal.

**Not file-bug.** A chore does not cause wrong behavior today. If it does, it's a bug.

## The Process

### Phase 1: Gather Context

YOU have the context - YOU describe it. Don't hand the writing back to the user.

Collect:
1. **Work** - What needs to change
2. **Timing** - What makes it worth doing now rather than never
3. **Completion** - How you would know it's done
4. **Evidence** - Files, PRs, or cards that show the current state

### Phase 2: Format Content

**Title:** a plain sentence describing the work. No kind prefix — the `chore` label carries the kind.

- Good: `Drop the unused legacy_sessions table`
- Bad: `Chore: legacy_sessions cleanup`

**Description template:**

```
## What
[The work, in one or two sentences]

## Why now
[What makes this worth doing now — cost of waiting, blocked work, upcoming change]

## Done when
[Observable completion condition]

## Notes
[Links and evidence only — see below]
```

**Gherkin is optional.** Use Given/When/Then in `## Done when` when the chore has
a behavioral edge worth pinning down; a plain observable condition is fine
otherwise ("Then the table is gone and no code references it").

**"Why now" must survive the question "what if we never did this?"** If the honest
answer is "nothing changes", don't file the card.

**Notes holds links and evidence only:** incident URL, PR, card, commit, file path.
If a Notes line does not change what someone does, cut it. It is not the
commentary field.

**Hard cap: 150 words for the whole description, Notes included.**

Over the cap means one of two things:

- It is more than one card → split it.
- The detail belongs in the design document → attach that file and link it from Notes.

**The design document is not yours to write.** It comes from the design
conversation that happened upstream — brainstorming, planning, whatever produced
it — and normally lives in `doc/scratchpad/` or `docs/plans/`. Attach the file
that already exists, as it is.

Do not author a design document to satisfy this step, and do not restate its
contents in the description. If no such document exists and the card still will
not fit in 150 words, it is more than one card. Split it.

Split cards go in priority order, lowest first, so the highest lands at the top
of Inbox. If the split work needs one place to track it, that is the parent-card
workflow — see create-parent-from-plan. Say so rather than collapsing the split.

### Phase 3: Create Card

```bash
bin/trello card new "<plain sentence describing the work>" \
  --description "<formatted description>" \
  --label chore \
  --position top
```

If attachments exist:
```bash
bin/trello attach upload <card-ref> <file>
```

### Phase 4: Confirm

Report ONLY: "Filed as #<number> - <short-url>"

Nothing more. Don't fill context with card details.

## Red Flags

If you catch yourself doing these, STOP:

- **Prefixing the title with "Chore:"** - The label carries the kind
- **Filing a bug as a chore** - If it causes wrong behavior today, use file-bug and the `bug` label
- **"Why now" that restates "What"** - "Because the table is unused" is not a reason to act now
- **Unverifiable "Done when"** - "Code is cleaner" can't be checked; name what an observer would see
- **Using Notes as commentary** - Links and evidence only; cut any line that doesn't change what someone does
- **Description over 150 words** - Split the card or attach the detail
- **Writing a design document to satisfy the attachment** - Attach what the design conversation produced; if nothing exists, split the card
- **Restating the attachment in the description** - Link it, don't duplicate it
- **Uploading a path that doesn't exist** - The attachment must exist before `attach upload`
- **Forgetting the chore label** - Every chore needs the label
- **Putting card in wrong place** - Top of Inbox, always
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect work, timing, completion condition | Context in memory |
| 2. Format | Apply template, Gherkin optional, ≤150 words | Description + files |
| 3. Create | Run CLI commands | Card created |
| 4. Confirm | Report reference | "Filed as #47" |
