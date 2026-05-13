---
name: Bee Workflow
description: Use when running the full Bee extraction workflow -- sync conversations, extract commitments/facts/risks, present for review, and optionally write back approved items.
icon: icon.svg
user_invocable: true
metadata:
  category: workflow/process
  family: bee
  lifecycle: active
  canonical_slug: bee-workflow
  icon_style: craft-category-glyph-v1
---

# Bee Workflow

Run the complete Bee extraction and review pipeline.

## When to use

- Daily review of new conversations
- Processing a specific conversation for action items
- Running the full detect -> propose -> review -> apply cycle

## Procedure

### 1. Check Prerequisites

```bash
curl -s --connect-timeout 3 http://127.0.0.1:8787/v1/me >/dev/null 2>&1 && echo "PROXY: OK" || echo "PROXY: DOWN"
curl -s --connect-timeout 3 http://localhost:3000/api/review >/dev/null 2>&1 && echo "APP: OK" || echo "APP: NOT_RUNNING"
```

If proxy is down, try FRIDAY: `curl -s http://friday-mac-local:8787/v1/me`
If app is not running, offer to start it: `cd ~/Projects/bee-workflow-os && pnpm dev`

### 2. Sync New Data

If the workflow app is running, use its sync endpoint. Otherwise use the CLI:
```bash
bee sync --output ~/.local/share/bee-sync/
```

### 3. Fetch Recent Conversations

```bash
bee conversations list --limit 10 --json
```

For each conversation, fetch details:
```bash
bee conversations get <id> --json
```

### 4. Extract Candidates

For each conversation, analyze the text (title + summary + transcript + utterances) for:

**Signal patterns (from bee-workflow-os extraction.ts):**

- **Commitments (todo)** — confidence 0.74:
  Phrases: "I will", "I'll", "we will", "need to", "follow up", "schedule",
  "send", "prepare", "set up", "look into", "get back to", "make sure"

- **Relationship Memory (fact)** — confidence 0.62:
  Phrases: "prefers", "likes", "dislikes", "works at", "lives in",
  "usually", "always", "never", "allergic", "birthday"

- **Decisions** — confidence 0.68:
  Phrases: "decided", "agreed", "approved", "we are going with",
  "the plan is", "we chose", "final decision"

- **Risks** — confidence 0.66:
  Phrases: "risk", "blocked", "blocker", "issue", "concern",
  "stuck", "delay", "problem", "warning"

- **Open Questions** — confidence 0.64:
  Phrases: "open question", "unclear", "waiting on", "need to confirm",
  "TBD", "not sure", "to be decided"

Filter: minimum 12 characters per candidate, deduplicate by normalized text.

### 5. Due Date Inference (for todos)

Parse date patterns from text:
- Direct: `2026-03-10`, `March 10`
- Relative: `tomorrow`, `next Friday`, `next week`
- Time: `at 3pm`, `noon`, `EOD`, `end of day`

### 6. Present for Review

Group items by kind and present:

## Extraction Results

### Commitments (N items)
| # | Text | Source | Confidence | Due |
|---|------|--------|------------|-----|

### Facts (N items)
| # | Text | Source | Confidence |

### Decisions (N items)
| # | Text | Source | Confidence |

### Risks (N items)
| # | Text | Source | Confidence |

### Open Questions (N items)
| # | Text | Source | Confidence |

### 7. Review and Act (REQUIRES USER CONFIRMATION)

For each item the user wants to act on:

**Approve (todo)** — PRIVACY GATE:
- Show: "Will create Bee todo: '<text>' [due: <date>]"
- Require explicit "yes"
- Then: `bee todos create --text "<text>" [--alarm-at <ISO>] --json`

**Approve (fact)** — PRIVACY GATE:
- Show: "Will create Bee fact: '<text>'"
- Require explicit "yes"
- Then: `bee facts create --text "<text>" --json`

**Dismiss** — Local only, no write to Bee.

**Snooze** — Local only, revisit later.

### 8. Summary

Report:
- Conversations processed: N
- Candidates extracted: N (by kind)
- Approved and written to Bee: N
- Dismissed: N
- Snoozed: N

$ARGUMENTS


## Pipeline

```
Intent → Authenticate Bee API → Extract/sync/audit → Structure output → Route artifacts
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Compact summary | Normal use |
| `full` | Complete data dump | Detailed analysis |
| `json` | Raw JSON | Programmatic use |

## Artifact Routing

- Bee data exports: `ShadowArchive/80-reports/bee-exports/`
- Audit reports: `ShadowArchive/80-reports/bee-audit-YYYY-MM-DD.md`

## Prerequisites

- Bee AI account with active session
- `bee` CLI or API access

## Contract

- Never expose Bee API tokens in output
- Respect Bee rate limits
- Export user data only with operator confirmation

