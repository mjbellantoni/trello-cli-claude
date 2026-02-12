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

### Phase 2: Format Content

**Description template:**

```
## Steps to Recreate
1. [What was being done]
2. [What action triggered it]

## Expected Behavior
[What should have happened]

## Actual Behavior
[What actually happened]

## Notes
[Any additional context - file paths, config, etc.]
```

**Attachments:**
- Stack traces > 10 lines → save to temp file, attach as `stacktrace.txt`
- Screenshots → attach as `screenshot.png`

### Phase 3: Create Card

```bash
bin/trello card new "Bug: <concise title>" \
  --description "<formatted description>" \
  --label bug \
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

- **Asking user to describe the bug** - You saw it, you describe it
- **Including full file contents** - Summarize, don't dump
- **Forgetting the bug label** - Every bug needs the label
- **Putting card in wrong place** - Top of Inbox, always
- **Verbose confirmation** - Just the reference, nothing more

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Gather | Collect error, steps, expected/actual | Context in memory |
| 2. Format | Apply template, handle attachments | Description + files |
| 3. Create | Run CLI commands | Card created |
| 4. Confirm | Report reference | "Filed as #47" |
