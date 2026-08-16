---
name: file-bug
description: Use when user says "file a bug for that", "create a ticket", "that's a bug - capture it", or wants to record an issue for later work
---

# File Bug

## Overview

Capture current session context into a Trello card so work can continue later. Returns just the card reference - keeps context minimal.

## The Process

### Phase 1: Gather Context

YOU saw the issue - YOU describe it. Don't ask the user what happened.

Collect:
1. **Error/Issue** - What went wrong (error message, unexpected behavior)
2. **Steps to recreate** - What was happening when it occurred
3. **Expected vs Actual** - What should have happened
4. **Stack trace** - If available
5. **Screenshot** - If Chrome connected and UI-related, capture it

### Phase 2: File the Card

```bash
bin/trello bug new "<plain sentence describing the defect>" \
  --steps "<step>" "<step>" "<step>" \
  --expected "<what should have happened>" \
  --actual "<what actually happened>" \
  --notes "<links and evidence only>"
```

You pass fields; the CLI applies the `bug` label, assembles the headings, puts
the card at the top of Inbox, and enforces the word cap. Don't build the
description and don't count words.

- **Don't number the steps.** `--steps` numbers them; a leading `1.` you type is stripped.
- **One `--steps` flag, several values after it** — or one value with newlines. Repeating the flag does not accumulate: `--steps "a" --steps "b"` silently keeps only `b`.
- `--notes` is optional.

**Notes holds links and evidence only:** incident URL, PR, card, commit, file path.
If a Notes line does not change what someone does, cut it. It is not the
commentary field.

**Attachments:**
- Stack traces > 10 lines → save to temp file, attach as `stacktrace.txt`
- Screenshots → attach as `screenshot.png`
- An incident or design document that already exists → attach that file as it is

```bash
bin/trello attach upload <card-ref> <file>
```

Stack traces and screenshots are session evidence, so you capture them yourself.
A prose document is not — attach the one that already exists rather than writing
one up. Either way, upload only after the file exists.

### Phase 3: Confirm

Report ONLY: "Filed as #<number> - <short-url>"

Nothing more. Don't fill context with card details.

## When the CLI Refuses the Card

The command exits non-zero and creates nothing — no card, and no HTTP request
even attempted. Over the cap, the error prints a per-field word breakdown, so
the oversized field is visible:

```
Error: bug description is 215 words, over the 200-word cap.
  steps 5 | expected 204 | actual 4 | notes 2
```

**The remedy is never to trim toward the limit.** Over the cap means one of two
things:

- It is more than one card → split it. File the splits in priority order, lowest first, so the highest lands at the top of Inbox.
- The detail belongs in an attachment → attach the file and link it from Notes.

There is no `--force`, and the caps are deliberately not adjustable from the
command line. The only override is `TRELLO_BUG_WORD_CAP` in `.trello.yml` — an
operator's decision about the board, not a move you make to get a card through.

## Red Flags

If you catch yourself doing these, STOP:

- **Asking user to describe the bug** - You saw it, you describe it
- **Rewording to slip under the cap** - Split the card or attach the detail; the cap is not a formatting target
- **Using Notes as commentary** - Links and evidence only; cut any line that doesn't change what someone does
- **Writing up a prose document to attach** - Capture stack traces and screenshots; attach existing documents as they are
- **Uploading a path that doesn't exist** - The attachment must exist before `attach upload`
- **Including full file contents** - Summarize, don't dump
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect error, steps, expected/actual | Context in memory |
| 2. File | Run `bug new`, upload attachments | Card created |
| 3. Confirm | Report reference | "Filed as #47" |
