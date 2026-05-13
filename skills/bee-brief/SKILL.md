---
name: Bee Brief
description: Use when generating today's Bee daily brief (calendar, email, todos, recent conversations). Use when the user runs /bee-brief or asks for a Bee daily brief. Includes auth check and robust handling of large bee now output.
icon: icon.svg
user_invocable: true
metadata:
  category: system/security
  family: bee
  lifecycle: active
  canonical_slug: bee-brief
  icon_style: craft-category-glyph-v1
---

# Bee Daily Brief

Generate today's daily brief from Bee: calendar, email, todos, recent conversations, and optional workflow inbox.

## When to use

- User runs `/bee-brief` or asks for "today's brief", "Bee brief", "daily brief from Bee".

## Procedure

### 0. Check auth first

Before fetching any data, verify Bee CLI is logged in:

```bash
bee me --json 2>/dev/null
```

If this fails or returns an error (e.g. "Not logged in. Run 'bee login' first"), **do not** run the remaining steps. Compose the brief with:

- A single **Bee status** block: "Bee CLI is not logged in. Run `bee login`."
- All other sections as "*No data — Bee not logged in.*"
- A short **Next steps**: run `bee login`, then re-run the brief.

Then stop. See bee-status skill if the user needs connectivity or proxy checks.

### 1. Fetch today (calendar + email)

```bash
bee today --json
```

### 2. Fetch daily summaries

```bash
bee daily list --limit 1 --json
```

### 3. Fetch open todos

```bash
bee todos list --json
```

### 4. Fetch recent conversations (last 24h)

**Prefer this to avoid truncated JSON:** `bee now --json` can be very large (full transcriptions). Pipe through `jq` to extract only what the brief needs, so output is not truncated when written to a file:

```bash
bee now --json 2>/dev/null | jq -c '.conversations[]? | {id, start_time, end_time, short_summary, summary}' 2>/dev/null
```

- Each line is one conversation (NDJSON). Parse line-by-line or collect into an array.
- Duration in minutes: `(end_time - start_time) / 60000`.
- Use `short_summary` as title; use `summary` for key points. Flag commitments/risks if present in summary.

If `jq` is not available or the pipeline fails, fall back to `bee now --json` and parse normally; if the result is written to a file and JSON parse fails (truncation), note in the brief that "Recent conversations (partial — output truncated)" and list what was parsed.

### 5. Optional — workflow inbox

```bash
curl -s http://localhost:3000/api/daily-brief 2>/dev/null
```

If empty or non-200, state "Workflow app not running" or "Enriched brief unavailable."

### 6. Compose the brief

- **## Daily Brief - [date]**
- **### Calendar & Email (from `bee today`)** — events with times, email summaries.
- **### Open Todos ([count])** — list todos; mark overdue; show alarm times where set.
- **### Recent Conversations ([count])** — title, duration, key point; flag commitments/risks if any.
- **### Daily Summary (from `bee daily`)** — Bee's short_summary and/or summary.
- **### Workflow Inbox (if app running)** — pending review, overdue, high-signal.

PRIVACY: Read-only. All processing local. No writes to Bee.

## See also

- **bee-status** §7 (Daily brief) — troubleshooting when brief is empty or `bee now` is large; `~/.agents/skills/bee-status/SKILL.md`.


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

