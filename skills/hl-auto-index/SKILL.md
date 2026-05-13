---
name: Headless Auto Index
description: Use when automating semantic index rebuilds from codebase changes, triggers, and scheduled runs.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-auto-index
  icon_style: craft-category-glyph-v1
---

# Auto Index

**Trigger**: Codebase changes, "rebuild index", "update search", manual indexing request

**Purpose**: Automated exploration and semantic indexing using shadow tech research.

## Core Pattern

Auto-index on:
- Session start (check for changes)
- Explicit trigger (`touch ~/.Codex/.trigger-index`)
- Scheduled runs (hourly via LaunchAgent)
- After major operations (git commits, large additions)

## Atomic Functions

### Trigger Index

```bash
# Trigger full index rebuild
touch ~/.Codex/.trigger-index

# Next session start will run full index
```

### Manual Index

```bash
# Check for changes
python3 /Volumes/☯Duality/ShadowArchive/10-projects/shadow-plugins/scripts/shadow-auto-indexer.py check

# Full index rebuild
python3 /Volumes/☯Duality/ShadowArchive/10-projects/shadow-plugins/scripts/shadow-auto-indexer.py index

# Index specific source
python3 /Volumes/☯Duality/ShadowArchive/10-projects/shadow-plugins/scripts/shadow-auto-indexer.py index /path/to/source
```

### LaunchAgent Automation

```bash
# Load agent (runs hourly)
launchctl load ~/Library/LaunchAgents/com.shadow.auto-indexer.plist

# Unload agent
launchctl unload ~/Library/LaunchAgents/com.shadow.auto-indexer.plist

# Check status
launchctl list | grep shadow.auto-indexer
```

## Change Detection

Uses SHA256 hash to detect changes:
- File modification times
- File count changes
- Directory structure updates

## Index Sources

Default sources indexed:
- `ShadowArchive/03-agent-roots/` — Agent code
- `ShadowArchive/10-projects/` — Shadow projects
- `spaces/` — Shadow spaces

## Rules

- **Index on change** — only rebuild if hash changed
- **Graceful degradation** — exit 0 if SHADOW drive missing
- **Timeout protection** — 300s max per index run
- **Background operation** — hooks run in parallel, don't block session

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 + Semantic Index + Hook resilience patterns.

**Atomic output**: Change detection + semantic index + registry state update.


## Pipeline

```
Intent → Authenticate browser session → Navigate to target → Extract/interact → Return structured output
```

1. Resolve target (URL, search query, operation)
2. Connect to authenticated browser session (Chrome profile)
3. Navigate and wait for page load
4. Extract data or perform interaction via Playwright/browser_tool
5. Return structured output (JSON, markdown, file)

## Modes

| Mode | Output | When |
|------|--------|------|
| `extract` (default) | Structured data extraction | Read/query operations |
| `interact` | Perform action, return result | Click, fill, submit |
| `monitor` | Continuous or periodic check | Watch, wait, alert |

## Artifact Routing

- Extracted data: presented in chat or saved to `ShadowArchive/80-reports/`
- Downloaded files: browser default download folder → classify → move to program structure

## Fallback Chain

1. Playwright MCP / browser_tool (primary)
2. Chrome DevTools Protocol via chrome-web-cli
3. AppleScript Chrome control (last resort)
4. If auth required and not available: stop and ask operator to open authenticated session

## Prerequisites

- Chrome with authenticated session for target service
- Playwright MCP tools or browser_tool available
- Network access to target service (VPN if enterprise)

## Error Handling

- **Auth wall / sign-in page**: Stop, inform operator, do not attempt credentials
- **Element not found**: Wait 3s, retry with relaxed selector, then report
- **Rate limited**: Back off 5s, retry once, then stop
- **VPN required**: Report "VPN down" and stop

## Contract

- **Never enter credentials** — assume authenticated session exists
- **Never click Send/Submit on final actions** (email, message) without operator approval
- **Respect rate limits** — 2s minimum between automated requests
- **Preserve browser state** — do not log out, close tabs, or clear cookies
- **Report auth failures** — do not silently skip locked content

