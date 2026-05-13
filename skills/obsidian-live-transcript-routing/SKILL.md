---
name: Obsidian Live Transcript Routing
description: Use when routing speak2 or Parakeet live transcripts into Obsidian using folder-specific deterministic transforms and optional Bee action sinks.
icon: icon.svg
metadata:
  category: memory/context
  family: obsidian
  lifecycle: active
  canonical_slug: obsidian-live-transcript-routing
  icon_style: craft-category-glyph-v1
---

# Obsidian Live Transcript Routing

Parakeet/Speak2 live transcription into Obsidian with folder-based deterministic transforms and optional Bee CLI actions. Use when building or updating a live transcript surface that tails Speak2 output and writes into Obsidian notes.

## When to Use
- Speak2 or Parakeet is the transcription source
- Obsidian is the primary live note surface
- Transcript behavior should vary by folder or note type
- Bee facts/todos are optional actions, not the primary storage surface

## Workflow
1. Treat Speak2 relay as append-only final utterances.
2. Read from `~/.speak2/transcriptions.jsonl` first.
3. Normalize transcript text lightly at parse time.
4. Resolve the most specific folder rule for the active note.
5. Render deterministic markdown by `mode`.
6. Replace only the managed section in the note, not the whole note.
7. Keep Bee writes explicit user actions, not automatic promotion.

## Source Contract
- Speak2 emits one JSON object per completed utterance.
- No partials, no replace/update semantics, no incremental stream state.
- Relay fields:
  - `ts`
  - `model`
  - `text`
  - `duration_ms`
  - `app`
- `duration_ms` is transcription latency, not utterance duration.

## Rule Model
Each rule should support:
- `folder`
- `heading`
- `instruction`
- `mode`

Most-specific folder wins.

Recommended modes:
- `daily` or `inbox`: checklist capture
- `meeting`: action items, decisions, open questions, transcript
- `project`: action log, next steps, decisions, open questions
- `timeline`: raw chronological transcript

## Deterministic Transform Heuristics
- Keep transforms local by default.
- Use lightweight classification only:
  - action lines: `need to`, `follow up`, `can you`, `update`, `todo`
  - decision lines: `decided`, `agreed`, `freeze`, `approved`, `we will`
  - question lines: trailing `?` or interrogative phrasing
- Preserve original wording in rendered sections.
- Do not hallucinate summaries or owners.

## Parakeet Optimization Rules
- Optimize for one-shot final lines, not partial replacement.
- Do not build a patch/update pipeline for Parakeet relay.
- Light cleanup only:
  - trim whitespace
  - collapse repeated spaces
  - strip obvious noise markers like `[music]`, `(laughter)`
  - keep wording otherwise intact
- Prefer file polling or file backfill first; socket can be additive later.

## Bee Positioning
- Obsidian is the live working surface.
- Bee CLI is a sink for approved actions:
  - `bee facts create --text ... --json`
  - `bee todos create --text ... --json`
- Do not auto-promote every transcript line into Bee.

## Gotchas
- Installed plugin `data.json` can override newer source defaults; keep deployed rules in sync.
- Missing `mode` fields cause silent fallback to timeline rendering.
- Polling the same JSONL repeatedly needs a latest-entry signature or equivalent change check.
- Do not use clipboard-based live capture when Speak2 already injected text and relayed the final utterance.
- Obsidian can drop a failing community plugin from `community-plugins.json` after startup failure. Re-check the enabled list after every failed load.
- Keep plugin boot thin. Avoid top-level `require("./bridge.cjs")` or other heavy imports if the bridge can be lazy-loaded after `onload()`.
- Avoid doing real vault reads or writes during `onload()`. Defer first sync and catch runtime read/write failures so the plugin can still load.
- Match local working plugin loader patterns: prefer plain builtin imports like `require("child_process")`, not `require("node:child_process")`, unless a working Obsidian plugin here already proves that form.

## Crystallized From
- Session: 2026-04-18 on JARVIS
- Original task: build an Obsidian plugin for live transcription using Speak2/Parakeet with Bee integration
- Extracted pattern: Parakeet final-utterance relay -> local deterministic rule engine -> Obsidian section sync -> optional Bee action sink
- Adjacent surfaces checked:
  - Speak2 source relay behavior
  - CASER STT relay reuse
  - existing Obsidian plugin scaffold
  - installed vault plugin state
  - Obsidian community plugin enable/disable behavior during failed boots
- Intentionally left untouched:
  - AI transform path
  - socket-first relay path
  - automatic Bee promotion


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

