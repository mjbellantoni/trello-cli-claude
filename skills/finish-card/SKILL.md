---
name: finish-card
description: Use when work is complete to move card to appropriate Done list based on whether code was committed
---

# Finish Card

## Overview

Complete a card by moving it to the appropriate Done list. The destination depends on whether the work involved committed code.

## List Semantics

- **Done/Committed**: Code was merged to main branch. Use when PRs are merged or commits are made.
- **Done/Deployed**: No code changes (research, planning, already deployed tasks)

## The Process

### Phase 1: Determine Destination

**Check for recent commits:**

```bash
git log --oneline --since="2 hours ago"
```

**Decision logic:**
- If commits relate to this card → `Done/Committed`
- If `/commit` skill was used in this session → `Done/Committed`
- If pure research/planning/discussion → `Done/Deployed`
- If unclear → Ask user

### Phase 2: Ask If Ambiguous

Use AskUserQuestion when unclear:

```
question: "Which Done list should this card move to?"
header: "Destination"
options:
  - label: "Done/Committed"
    description: "Code was committed/merged to main branch"
  - label: "Done/Deployed"
    description: "No code changes or already deployed"
```

### Phase 3: Move Card

```bash
bin/trello card move <ref> "<list-name>"
```

### Phase 4: Confirm

Report ONLY: "Moved #<number> to <list-name>"

## Red Flags

If you catch yourself doing these, STOP:

- **Guessing without checking git** - Always check for commits first
- **Moving incomplete work** - Verify work is actually done
- **Using wrong list for commits** - If code was committed, it's Done/Committed
- **Verbose confirmation** - Just the reference and list name
- **Forgetting to parse card reference** - Extract number or short link from context

## Quick Reference

| Scenario | Destination |
|----------|-------------|
| PR merged to main | Done/Committed |
| Used `/commit` in session | Done/Committed |
| Code changes committed | Done/Committed |
| Research/documentation only | Done/Deployed |
| Question answered | Done/Deployed |
| Already deployed/no action | Done/Deployed |
| Unclear | Ask user |
