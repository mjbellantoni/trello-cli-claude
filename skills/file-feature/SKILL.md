---
name: file-feature
description: Use when user says "file a feature for that", "make a card for this idea", "capture this as a feature", or wants to record a new user-facing capability for later work
---

# File Feature

## Overview

Capture a wanted capability into a Trello card so work can continue later. Returns just the card reference - keeps context minimal.

## The Process

### Phase 1: Gather Context

YOU have the context - YOU describe it. Don't hand the writing back to the user.

Collect:
1. **Capability** - What the user should be able to do that they can't today
2. **Motivation** - Who wants it and what it unblocks
3. **Acceptance** - How you would know it works, in observable terms
4. **Evidence** - Conversation, incident, card, or file that prompted it

### Phase 2: Format Content

**Title:** a plain sentence describing the capability. No kind prefix — the `feature` label carries the kind.

- Good: `Let reviewers filter the queue by assignee`
- Bad: `Feature: queue filtering`

**Description template:**

```
## What
[The capability, in one or two sentences]

## Why
[Who wants it and what it unblocks]

## Done when
Given <context>
When <action>
Then <observable outcome>

## Notes
[Links and evidence only — see below]
```

**Gherkin is required.** Every feature card gets at least one Given/When/Then in
`## Done when`. `Then` must name something observable — a screen state, a
response, a record, a message. "Then it works" is not an outcome.

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
bin/trello card new "<plain sentence describing the capability>" \
  --description "<formatted description>" \
  --label feature \
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

- **Prefixing the title with "Feature:"** - The label carries the kind
- **Skipping the Gherkin** - Given/When/Then is required on every feature card
- **Unobservable `Then`** - "Then it works" or "Then it's faster" can't be checked; name the screen state, response, record, or message
- **Writing a solution instead of a capability** - What the user can do, not which class you'd add
- **Using Notes as commentary** - Links and evidence only; cut any line that doesn't change what someone does
- **Description over 150 words** - Split the card or attach the detail
- **Writing a design document to satisfy the attachment** - Attach what the design conversation produced; if nothing exists, split the card
- **Restating the attachment in the description** - Link it, don't duplicate it
- **Uploading a path that doesn't exist** - The attachment must exist before `attach upload`
- **Forgetting the feature label** - Every feature needs the label
- **Putting card in wrong place** - Top of Inbox, always
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect capability, motivation, acceptance | Context in memory |
| 2. Format | Apply template, Gherkin required, ≤150 words | Description + files |
| 3. Create | Run CLI commands | Card created |
| 4. Confirm | Report reference | "Filed as #47" |
