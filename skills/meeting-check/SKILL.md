---
name: 'Meeting Context Check'
description: 'Use when the user needs: Fast meeting context — check calendar, find recent/upcoming meetings, extract recaps, transcripts, action items. Fastest path to meeting intel.'
icon: icon.svg
autoTrigger: true
triggers:
- check meeting
- recent meeting
- what meeting
- meeting context
- meeting recap
- today's meetings
- calendar
metadata:
  depends_on: [shadow-lists]
  category: enterprise/astemo
  family: dev
  lifecycle: active
  canonical_slug: meeting-check
  icon_style: craft-category-glyph-v1
---

# Meeting Check — Intelligent Agent Dispatch

## When triggered, execute this sequence:

### Step 1: Identify (5 seconds)
```
owl_brief → get Bee's latest conversations
```
- The MOST RECENT completed conversation = the meeting that just happened
- Match Bee conversation time against the meeting registry (see AGENTS.md)
- If Bee has a live `CAPTURING` conversation → meeting is happening now

### Step 2: Correct (2 seconds)
Cross-reference ALL names in Bee output against `kernel/vocabulary.md`:
- Weiss-Aug (not Wysock)
- Nyssa (not Nisa)
- FH4L (not FH-IVL)
- XY9H (not XY-NINE-H)

### Step 3: Enrich (10 seconds)
If Bee summary is thin, check secondary sources:
```
tap_execute → Teams Chat → find meeting chat → read recent messages
```
Or if user recorded with Google Recorder:
```
tap_execute → recorder.google.com → find by timestamp → extract transcript
```

**Recorder identity boundary:**
- Google Recorder is locked to `sdluffy` / `Default` / `sabarish.duvvuru@gmail.com`.
- Do **not** probe `sduvvuru` / Profile 5 or other Chrome profiles for Recorder because meeting topic sounds like work.
- If the operator already asked for Recorder specifically, do Recorder first. Do **not** widen into Bee/Teams/source-hopping unless Recorder is missing or the operator asks for broader context.

### Step 4: Merge & Anchor (5 seconds)
After extracting from any source, run the merge pipeline to create a durable anchor:
```
bash → python3 /Volumes/☯Duality/ShadowArchive/10-projects/shadow-plugins/scripts/meeting-capture-merge.py --date YYYY-MM-DD
```
This writes to `~/.shadow-research/queue/kernel-diff.jsonl` and deduplicates across runs.
Skip if `--dry-run` was already used and operator hasn't asked to persist.

### Step 5: Extract & Push (10 seconds)
- Parse action items from Bee summary
- Assign program (FH4S/FH4/XP7G/VEC/XY9H) using keyword detection
- Assign priority (P0 if overdue/red, P1 if this week, P2 if this month)
- **Primary: Push to SHADOW-Agent-Tasks list** (see `shadow-lists` skill)
- Fallback: Push to Smartsheet `fh4s_actions` (sheet 8085886420864900) if list API unavailable

**List push workflow:**
1. Ensure browser is on `astemogroup.sharepoint.com` (any page)
2. Cache digest: `evaluate (async()=>{const r=await fetch('/sites/USGRPxEV/_api/contextinfo',{method:'POST',headers:{Accept:'application/json;odata=verbose'}});const d=await r.json();window._d=d.d.GetContextWebInformation.FormDigestValue;return 'OK'})()`
3. Create task per action item: `evaluate` with POST to `/sites/USGRPxEV/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/items`
4. Use entity type: `SP.Data.SHADOWAgentTasksListItem`
5. Fields: Title, Program, TaskType, Status='queued', Priority, Source='meeting', SourceRef=meeting title, Owner

See `shadow-lists` WORKFLOWS.md for copy-paste snippets (SL_CREATE, SL_MEETING).

### Step 6: Report
Present as clean table:
```
Meeting: [title] | [date] [time] | [program]
Source: [bee/recorder/teams] | [utterances] utterances

Action Items:
| # | Item | Owner | Priority | Due |
|---|------|-------|----------|-----|
```

## Meeting Registry (from AGENTS.md)
| Meeting | Day | Time | Program |
|---------|-----|------|---------|
| Honda INV DE Weekly | Mon | 9:00 AM | Cross |
| Honda Gen4 VA PCU CFT | Mon | 11:00 AM | FH4S |
| FH4S IPM DE Weekly | Tue | 6:00 PM | FH4S |
| Gen4-VA GCFT | Wed/Thu | varies | FH4S |
| IPM Localization GCFT | Thu | varies | FH4S |
| Gen4VA Tariff Mitigation | Thu | 7:00 AM | FH4S |
| Gen4VA CFT (Miki) | Thu | varies | FH4S |
| INV VEC Updates | Fri | 11:00 AM | VEC |
| 1-on-1 (SD, KB) | Fri | varies | Internal |

## Source Priority
1. `owl_brief` (Bee AI) — ALWAYS check first
2. Google Recorder — if user mentions "I recorded it"
3. Teams Recap — if Teams meeting with recording
4. Teams Chat — persistent messages
5. Local files — LAST resort

**Correction override:**
- If the operator corrects the source or profile once, stop re-ranking source priority from defaults. Use the corrected lane only.

## Pipeline

```
Identify (owl_brief → most recent Bee conversation → match meeting registry)
  ↓
Correct (cross-reference names against kernel/vocabulary.md)
  ↓
Enrich (Bee summary → Teams Chat / Google Recorder if thin)
  ↓
Merge & Anchor (meeting-capture-merge.py → kernel-diff.jsonl)
  ↓
Extract & Push (parse actions → assign program/priority → push to SHADOW-Agent-Tasks list)
  ↓
Route
  ├── Compact meeting report (default chat output)
  ├── Action items → SHADOW-Agent-Tasks SharePoint list
  ├── Merge artifacts → ~/.shadow-research/queue/
  └── Durable anchor → ShadowArchive/80-reports/
```

## Modes

| Mode | Trigger | Output |
|------|---------|--------|
| `check` / default | "check meeting", "recent meeting", "what meeting" | Compact meeting report with action table |
| `live` | "live meeting", "meeting happening" | Real-time capture if Bee shows CAPTURING state |
| `recap` | "meeting recap", "what happened" | Full recap with enriched context |
| `actions` | "action items", "what do I owe" | Extracted action items only |
| `recorder` | "I recorded it", "Google Recorder" | Google Recorder transcript extraction first |
| `push` | "push actions", "sync tasks" | Force push action items to SharePoint list |

Default output is compact: meeting title, date/time, program, source, action table.

## Artifact Routing

| Artifact | Path | Notes |
|----------|------|-------|
| Meeting report | `ShadowArchive/80-reports/meeting-YYYY-MM-DD-<title>.md` | Durable meeting extract |
| Action items | SHADOW-Agent-Tasks SharePoint list | Primary push target |
| Action items (fallback) | Smartsheet `fh4s_actions` (sheet 8085886420864900) | Fallback if list API unavailable |
| Merge artifacts | `~/.shadow-research/queue/kernel-diff.jsonl` | Research pipeline feed |

## Fallback Chain

```
1. owl_brief (Bee AI)  (primary — ALWAYS check first)
   ↓ (Bee unavailable / no recent conversation)
2. Google Recorder  (if operator mentions "I recorded it")
   ↓ (no recording / Recorder locked to wrong profile)
3. Teams Recap  (AI summary from recorded Teams meeting)
   ↓ (no recording / Teams Recap unavailable)
4. Teams Chat  (persistent messages from meeting thread)
   ↓ (no chat / external meeting)
5. Local files  (last resort — meeting notes in program directory)
```

Always disclose which source was used. If operator corrects source priority once, use corrected lane only.

## Prerequisites

- **Bee AI** connected and synced — `owl_brief` available for conversation capture
- **kernel/vocabulary.md** accessible — for name correction (Weiss-Aug, Nyssa, FH4L, XY9H)
- **meeting-capture-merge.py** at `ShadowArchive/10-projects/shadow-plugins/scripts/` — for merge pipeline
- **SharePoint list access** — `SHADOW-Agent-Tasks` on `astemogroup.sharepoint.com/sites/USGRPxEV`
- **Browser on SharePoint domain** — required for list push digest caching

## Error Handling

| Failure | Recovery |
|---------|----------|
| Bee unavailable | Skip Step 1; start from Teams/Recorder sources |
| Name misrecognized | Cross-reference against kernel/vocabulary.md; flag uncertain matches |
| SharePoint list push fails | Fall back to Smartsheet `fh4s_actions`; retry list later |
| Google Recorder locked to wrong profile | Do NOT probe other Chrome profiles; ask operator to share transcript |
| Meeting not in registry | Classify by keywords (FH4S/VEC/MLC); ask operator to confirm program |
| Merge script fails | Save raw capture; retry merge later |
| Browser not on SharePoint domain | Navigate to any SharePoint page first; cache digest; then push |

## Contract

- **ALWAYS** check Bee first (`owl_brief`) — never skip to secondary sources.
- **Do not** probe Chrome profiles other than the operator's confirmed profile for Google Recorder.
- **Correct names** against kernel/vocabulary.md — never propagate misspellings.
- **Do not** expose meeting action items or decisions in customer-facing artifacts.
- **Do not** auto-commit to customer gates/deadlines from meeting captures — flag as `needs evidence`.
- **Correction override** — if operator corrects source priority once, stop re-ranking from defaults.
- Meeting captures are **confidential Astemo IP** — classification metadata only in training pairs.
