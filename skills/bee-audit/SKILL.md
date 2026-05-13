---
name: Bee Audit
description: 'Use when the user needs: Deep privacy audit of Bee AI data -- inventory all stored data, assess exposure, check connectors, and recommend deletions.'
icon: icon.svg
user_invocable: true
metadata:
  category: system/security
  family: bee
  lifecycle: active
  canonical_slug: bee-audit
  icon_style: craft-category-glyph-v1
---

# Bee Privacy Audit

Deep analysis of what Bee knows about you and where your data lives.

## When to use

- Periodic privacy check (weekly/monthly)
- Before connecting new services
- After a sensitive conversation was captured
- When deciding what to delete

## Procedure

### 1. Profile Assessment

```bash
bee me --json
```

Document: user ID, email, timezone, account creation date.
Note what PII Bee has on file.

### 2. Full Data Inventory

Enumerate every entity stored in Bee cloud:

```bash
# Count all data types
bee facts list --limit 500 --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
items = d if isinstance(d,list) else d.get('items',[])
confirmed = sum(1 for f in items if f.get('confirmed'))
print(f'Facts: {len(items)} total ({confirmed} confirmed, {len(items)-confirmed} unconfirmed)')
"

bee todos list --limit 500 --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
items = d if isinstance(d,list) else d.get('items',[])
completed = sum(1 for t in items if t.get('completed'))
print(f'Todos: {len(items)} total ({completed} completed, {len(items)-completed} open)')
"

bee conversations list --limit 500 --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
items = d if isinstance(d,list) else d.get('items',[])
print(f'Conversations: {len(items)} total')
"

bee daily list --limit 500 --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
items = d if isinstance(d,list) else d.get('items',[])
print(f'Daily summaries: {len(items)} total')
"

bee journals list --limit 500 --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
items = d if isinstance(d,list) else d.get('items',[])
print(f'Journals: {len(items)} total')
"
```

### 3. Unconfirmed Facts Review

These are facts Bee inferred without user approval:
```bash
bee facts list --unconfirmed --json
```

For each unconfirmed fact, display the text and recommend:
- Confirm if accurate and useful
- Delete if inaccurate or sensitive

### 4. Sensitive Content Scan

For each conversation, check for sensitive keywords:
- Financial: "salary", "bank", "payment", "credit", "SSN"
- Health: "doctor", "diagnosis", "medication", "therapy"
- Legal: "lawyer", "contract", "NDA", "confidential"
- Personal: "password", "secret", "private"

```bash
bee search --query "salary OR bank OR payment OR password OR confidential" --limit 20 --json
```

Flag any conversations containing sensitive content.

### 5. Local Data Footprint

```bash
# Synced data
du -sh ~/.local/share/bee-sync/ 2>/dev/null || echo "No sync dir"

# Workflow database
ls -lh ~/Projects/bee-workflow-os/data/bee-workflow-os.db 2>/dev/null || echo "No workflow DB"

# CLI credentials
security find-generic-password -s "bee-cli" 2>&1 | grep -c "password" && echo "CLI credentials in keychain" || echo "No CLI credentials"
```

### 6. Network Exposure Assessment

Check what services the proxy is exposed to:
```bash
# Local proxy binding
lsof -i :8787 2>/dev/null | head -5

# FRIDAY proxy
ssh -o ConnectTimeout=5 -o BatchMode=yes friday-mac-local 'lsof -i :8787 2>/dev/null | head -5' 2>/dev/null
```

Verify proxy is NOT exposed to the internet (should bind to localhost only).

### 7. Data Flow Map

Present the complete data flow:

```
Bee Band (mic) --BLE--> iPhone (Bee app)
  |
  v
Bee Cloud (Amazon/Google Cloud AI)
  - Audio: processed real-time, NOT stored (per privacy policy)
  - Transcripts: stored as conversations
  - Facts: auto-extracted, some unconfirmed
  - Todos: user-created or approved
  |
  v
Bee Proxy (localhost:8787 or FRIDAY:8787)
  - Authenticated local access to Bee API
  - No data cached by proxy
  |
  v
Local Processing (JARVIS/FRIDAY)
  - bee-workflow-os: SQLite DB with review items
  - bee sync: markdown exports
  - All extraction runs locally
```

### 8. Privacy Score and Recommendations

Calculate a simple privacy score based on:
- Unconfirmed facts count (fewer = better)
- iOS permissions denied (more denied = better)
- Proxy exposure (localhost only = good)
- Sensitive conversations flagged (fewer = better)

Recommendations:
1. Delete unconfirmed facts you don't want: `/bee-facts delete <id>`
2. Review and delete sensitive conversations if possible
3. Ensure iOS Bee permissions are minimal (mic + bluetooth only)
4. Verify proxy only binds to localhost
5. Run this audit periodically: `/bee-audit`

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

