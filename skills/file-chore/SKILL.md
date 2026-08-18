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

### Phase 2: File the Card

```bash
bin/trello chore new "<plain sentence describing the work>" \
  --what "<the work, in one or two sentences>" \
  --why-now "<what makes this worth doing now>" \
  --done-when "<observable completion condition>" \
  --notes "<links and evidence only>"
```

You pass fields; the CLI applies the `chore` label, assembles the headings, puts
the card at the top of Inbox, and enforces the word cap. Don't build the
description and don't count words.

- **One `--done-when` flag, several values after it** — or one value with newlines. This form works on every CLI version. As of trello-cli 2.9.0 repeating the flag (`--done-when "a" --done-when "b"`) accumulates too, but older versions keep only the last value, so stay with the single-flag form.
- `--notes` is optional.

**"Why now" must survive the question "what if we never did this?"** If the honest
answer is "nothing changes", don't file the card.

**Gherkin is optional.** Use Given/When/Then in `--done-when` when the chore has
a behavioral edge worth pinning down; a plain observable condition is fine
otherwise ("The table is gone and no code references it").

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
Error: chore description is 156 words, over the 150-word cap.
  what 9 | why_now 134 | done_when 11 | notes 2
```

**The remedy is never to trim toward the limit.** Over the cap means one of two
things:

- It is more than one card → split it. File the splits in priority order, lowest first, so the highest lands at the top of Inbox. If the split work needs one place to track it, that is the parent-card workflow — see create-parent-from-plan. Say so rather than collapsing the split.
- The detail belongs in the design document → attach that file and link it from Notes.

There is no `--force`, and the caps are deliberately not adjustable from the
command line. The only override is `TRELLO_CHORE_WORD_CAP` in `.trello.yml` — an
operator's decision about the board, not a move you make to get a card through.

## Red Flags

If you catch yourself doing these, STOP:

- **Filing a bug as a chore** - If it causes wrong behavior today, use file-bug and the `bug` label
- **"Why now" that restates "What"** - "Because the table is unused" is not a reason to act now
- **Unverifiable "Done when"** - "Code is cleaner" can't be checked; name what an observer would see
- **Rewording to slip under the cap** - Split the card or attach the detail; the cap is not a formatting target
- **Using Notes as commentary** - Links and evidence only; cut any line that doesn't change what someone does
- **Writing a design document to satisfy the attachment** - Attach what the design conversation produced; if nothing exists, split the card
- **Restating the attachment in the description** - Link it, don't duplicate it
- **Uploading a path that doesn't exist** - The attachment must exist before `attach upload`
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect work, timing, completion condition | Context in memory |
| 2. File | Run `chore new`, upload attachments | Card created |
| 3. Confirm | Report reference | "Filed as #47" |
