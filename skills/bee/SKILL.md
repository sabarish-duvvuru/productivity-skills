---
name: Bee
description: 'Use when the user needs: Bee AI ambient intelligence — status, sync, extraction, brief, search, facts/todos, stream, audit, privacy, setup. Use when the user mentions "Bee", "ambient capture", "meeting capture", "commitments", "what did I say", "sync bee", "bee status", "bee facts", "bee todos", "bee search", "bee audit", "bee brief", "bee privacy", or any Bee AI operation.'
icon: icon.svg
metadata:
  depends_on: [gli]
  category: content/intel
  family: bee
  lifecycle: active
  canonical_slug: bee
  icon_style: craft-category-glyph-v1
---

# Bee AI Operations

| Op | How |
|----|-----|
| **Status** | `owl.brief` via shadow-mcp, or `bee status` CLI |
| **Sync** | `bee sync` — pulls conversations locally |
| **Extract** | `owl.extract` with conversation ID — commitments, facts, risks |
| **Brief** | `owl.brief` — today's conversations, commitments, facts |
| **Search** | `bee search <query>` — keyword or neural |
| **Facts** | `bee facts list/create/update/delete` |
| **Todos** | `bee todos list/create/update/delete` |
| **Stream** | `bee stream` — real-time event monitoring |
| **Audit** | Full privacy inventory — stored data, connectors, deletion options |
| **Setup** | CLI install, login, FRIDAY proxy config, end-to-end verify |

Quality hierarchy: Google Recorder > Bee high-quality > Bee ambient > manual notes.

## Host Reality

- On JARVIS, Bee CLI is currently reachable at `~/.local/bin/bee` (expanded: `/Users/sdluffy/.local/bin/bee`).
- On FRIDAY, Bee CLI is currently reachable over SSH at `/opt/homebrew/bin/bee`.
- If mesh wrappers like `gli` are unavailable in the current shell, fall back to direct `ssh friday "bee ..."` for FRIDAY-side checks.


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

