---
name: Bookmark Obsidian
description: Use when saving the current Codex chat session as a bookmark in the Sessions Obsidian vault with CLI management tools
icon: icon.svg
metadata:
  category: memory/context
  family: memory
  lifecycle: active
  canonical_slug: bookmark-obsidian
  icon_style: craft-category-glyph-v1
---

# Bookmark Obsidian

## Overview

Saves the current chat session as a bookmark in your Obsidian Sessions vault (`~/clawd/obsidian/Sessions`) with full CLI management capabilities.

**Announce at start:** "I'm using the bookmark-obsidian skill to save this session."

## What It Does

1. Creates a new session note with timestamp and topic summary
2. Updates the Sessions Index with the new entry
3. Auto-sorts the index by recency (newest first)
4. Adds tags for easy searching
5. Supports starring (⭐) to pin important sessions to top
6. CLI tool for managing sessions outside of chat

## Vault Structure

```
~/clawd/obsidian/Sessions/
├── Sessions Index.md          # Main table of all sessions
├── Templates/
│   ├── Session Template.md    # Full session template
│   └── Quick Session.md       # Quick note template
├── Dashboard/
│   ├── Dashboard.md           # Dataview dashboard
│   └── Quick Reference.md     # CLI reference
├── Archive/                   # Archived sessions
└── 2026-01-28 - Session.md    # Session notes
```

## Using the CLI Tool

The `bookmark` CLI tool provides session management outside of chat:

```bash
# List all sessions
bookmark list

# Open a session in your editor
bookmark open "USB Drive"

# Search sessions
bookmark search "git"

# Star/unstar a session
bookmark star "USB Drive"

# Archive old sessions
bookmark archive "Old Session"

# Show statistics
bookmark stats

# Show recent sessions
bookmark recent 10

# List all tags
bookmark tags

# Open vault in Obsidian
bookmark vault
```

## Creating a Session Bookmark

### 1. Summarize the Session

Review the conversation and create a concise title.

Examples:
- "USB Drive Analysis and Setup"
- "Git Worktree Implementation"
- "API Endpoint Debugging"

### 2. Generate Timestamp

```bash
date +"%Y-%m-%d %H:%M"
```

### 3. Create Session Note

Create file at `~/clawd/obsidian/Sessions/YYYY-MM-DD - Session Title.md`:

```markdown
---
date: YYYY-MM-DD HH:MM
title: Session Title
status: completed
source: CLI
tags: [session, topic-tag]
starred: false
related: []
---

# Session Title

**Date:** YYYY-MM-DD HH:MM
**Source:** CLI
**Topic:** Brief one-line summary

## Summary

2-3 sentence overview of what was discussed and accomplished.

## Key Points

- Point 1
- Point 2
- Point 3

## Commands Used

\`\`\`bash
# Relevant commands
\`\`\`

## Files Modified

| File | Action |
|------|--------|
| path/to/file | Modified |

## Related Sessions

- [[Related Session Name]]

## Next Steps

- [ ] Action item 1
- [ ] Action item 2

---
**Metadata**
- Duration: ~30 minutes
- Model: Codex-opus-4-5
- Working Directory: `~/path/to/project`
```

### 4. Update Index

Read `~/clawd/obsidian/Sessions/Sessions Index.md` and add entry:

```markdown
| Star | Date | Source | Session | Topic | Status |
|------|------|--------|---------|-------|--------|
| ⭐ | 2026-01-28 | CLI | [Session Title](link.md) | Summary | completed |
| | 2026-01-27 | IDE | [Another](link.md) | Summary | completed |
```

**Sorting:** Starred first, then by date (newest first)

### 5. Report Success

```
Session bookmarked: "Session Title"
Location: ~/clawd/obsidian/Sessions/YYYY-MM-DD - Session Title.md
Index updated with newest entry first
Source: CLI
Use 'bookmark list' to see all sessions
```

## Quick Reference

| Action | Command |
|--------|---------|
| List sessions | `bookmark list` |
| Search | `bookmark search <term>` |
| Star session | `bookmark star <name>` |
| Open session | `bookmark open <name>` |
| Recent sessions | `bookmark recent [n]` |
| Statistics | `bookmark stats` |
| Open vault | `bookmark vault` |
| Get timestamp | `date +"%Y-%m-%d %H:%M"` |

## Sources

| Source | Description |
|--------|-------------|
| `CLI` | Codex terminal |
| `IDE` | VS Code / JetBrains extension |
| `Browser` | Codex.ai website |
| `Other` | API, MCP, custom tools |

## Dashboard (Obsidian)

Install the **Dataview** plugin in Obsidian to enable the Dashboard at `Dashboard/Dashboard.md` with:

- Recent sessions table
- Starred sessions
- Statistics by source
- Tag cloud

## Example Workflow

```
You: I'm using the bookmark-obsidian skill to save this session.

[Summarize: "USB Drive Analysis and Setup"]
[Get timestamp: "2026-01-28 20:30"]
[Detect source: CLI]
[Create note: 2026-01-28 - USB Drive Analysis.md]
[Update Sessions Index.md with new entry]

Session bookmarked: "USB Drive Analysis and Setup"
Location: ~/clawd/obsidian/Sessions/2026-01-28 - USB Drive Analysis.md
Index updated with newest entry first

Later, from terminal:
$ bookmark search "USB"
  Found: 2026-01-28 - USB Drive Analysis.md
$ bookmark star "USB Drive"
  Starred: USB Drive Analysis

$ bookmark stats
  Total Sessions:     42
  Starred Sessions:   5
  Archived Sessions:  3
```

## Opening the Vault

### Obsidian
1. Open Obsidian
2. Click "Open folder as vault"
3. Navigate to `~/clawd/obsidian/Sessions`

### Command Line
```bash
# Open in Obsidian
bookmark vault

# Or directly
open -a "Obsidian" ~/clawd/obsidian/Sessions
```

## Files Created

| File | Purpose |
|------|---------|
| `~/clawd/bin/bookmark` | CLI management tool |
| `Sessions/Sessions Index.md` | Main index table |
| `Sessions/Templates/*.md` | Note templates |
| `Sessions/Dashboard/*.md` | Obsidian dashboard |
| `.Codex/skills/bookmark-obsidian/SKILL.md` | This skill |


## Pipeline

```
Intent → Resolve vault path → Read/create/search note → Apply wikilinks/properties → Route
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Formatted note | Normal use |
| `raw` | Raw markdown | Programmatic processing |

## Artifact Routing

- Vault notes: written directly to configured Obsidian vault
- Session bookmarks: vault `Sessions/` folder

## Prerequisites

- Obsidian CLI available (`obsidian-cli`)
- Vault path configured

## Contract

- Preserve existing wikilinks and embeds
- Never overwrite without confirmation
- Follow vault folder conventions

