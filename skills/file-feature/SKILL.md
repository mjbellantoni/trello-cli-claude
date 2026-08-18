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

### Phase 2: File the Card

```bash
bin/trello feature new "<plain sentence describing the capability>" \
  --what "<the capability, in one or two sentences>" \
  --why "<who wants it and what it unblocks>" \
  --done-when "Given <context>" "When <action>" "Then <observable outcome>" \
  --notes "<links and evidence only>"
```

You pass fields; the CLI applies the `feature` label, assembles the headings,
puts the card at the top of Inbox, and enforces the word cap. Don't build the
description and don't count words.

- **One `--done-when` flag, several values after it** — or one value with newlines. This form works on every CLI version. As of trello-cli 2.9.0 repeating the flag (`--done-when "a" --done-when "b"`) accumulates too, but older versions keep only the last value, so stay with the single-flag form.
- `--notes` is optional.

**`Then` must name something observable** — a screen state, a response, a record,
a message. The CLI only checks that the words Given, When and Then are present;
it cannot tell whether the outcome can be seen. "Then it works" is not an
outcome.

**Notes holds links and evidence only:** incident URL, PR, card, commit, file path.
If a Notes line does not change what someone does, cut it. It is not the
commentary field.

**The design document is not yours to write.** It comes from the design
conversation that happened upstream — brainstorming, planning, whatever produced
it — and normally lives in `doc/scratchpad/` or `docs/plans/`. Attach the file
that already exists, as it is:

```bash
bin/trello attach upload <card-ref> <file>
```

Do not author a design document to satisfy this step, and do not restate its
contents in the description. If no such document exists and the card still will
not fit, it is more than one card. Split it.

### Phase 3: Confirm

Report ONLY: "Filed as #<number> - <short-url>"

Nothing more. Don't fill context with card details.

## When the CLI Refuses the Card

The command exits non-zero and creates nothing — no card, and no HTTP request
even attempted. Over the cap, the error prints a per-field word breakdown, so
the oversized field is visible:

```
Error: feature description is 161 words, over the 150-word cap.
  what 9 | why 8 | done_when 142 | notes 2
```

**The remedy is never to trim toward the limit.** Over the cap means one of two
things:

- It is more than one card → split it. File the splits in priority order, lowest first, so the highest lands at the top of Inbox. If the split work needs one place to track it, that is the parent-card workflow — see create-parent-from-plan. Say so rather than collapsing the split.
- The detail belongs in the design document → attach that file and link it from Notes.

There is no `--force`, and the caps are deliberately not adjustable from the
command line. The only override is `TRELLO_FEATURE_WORD_CAP` in `.trello.yml` —
an operator's decision about the board, not a move you make to get a card
through.

## Red Flags

If you catch yourself doing these, STOP:

- **Unobservable `Then`** - "Then it works" or "Then it's faster" can't be checked; name the screen state, response, record, or message
- **Writing a solution instead of a capability** - What the user can do, not which class you'd add
- **Rewording to slip under the cap** - Split the card or attach the detail; the cap is not a formatting target
- **Using Notes as commentary** - Links and evidence only; cut any line that doesn't change what someone does
- **Writing a design document to satisfy the attachment** - Attach what the design conversation produced; if nothing exists, split the card
- **Restating the attachment in the description** - Link it, don't duplicate it
- **Uploading a path that doesn't exist** - The attachment must exist before `attach upload`
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect capability, motivation, acceptance | Context in memory |
| 2. File | Run `feature new`, upload attachments | Card created |
| 3. Confirm | Report reference | "Filed as #47" |
