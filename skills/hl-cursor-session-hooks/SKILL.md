---
name: Headless Cursor Session Hooks
description: 'Use when the user needs: Capture Cursor session hook events into session logs for live prepaid tracking and attribution.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-cursor-session-hooks
  icon_style: craft-category-glyph-v1
---

# hl-cursor-session-hooks

**Pattern**: Cursor Composer/Agent hooks → session-log.jsonl for live prepaid tracking

## When to Use

- Tracking actual Cursor Pro usage in real-time
- Measuring prepaid credit utilization between quota resets
- Automating session logging without manual CLI calls

## Pattern

**Hook integration points:**

Cursor fires events at specific points. We want to capture:
- **Session start** → Create session record
- **Task completion** → Log task with provider attribution
- **Session end** → Finalize session record

**Session log format (JSONL):**

```json
{
  "session_id": "2026-04-19T21:15:30.123456",
  "start_time": "2026-04-19T21:15:30.123456",
  "tasks": [
    {
      "task_type": "editor_native_coding",
      "provider": "cursor-pro",
      "tokens": 1234,
      "cost_usd": 0.05,
      "model": "composer-2-fast",
      "timestamp": "2026-04-19T21:15:30.123456"
    }
  ],
  "total_tokens": 1234,
  "provider_usage": {
    "cursor-pro": {
      "tokens": 1234,
      "cost_usd": 0.05,
      "task_count": 1
    }
  },
  "end_time": "2026-04-19T21:15:35.123456",
  "duration_seconds": 5.0
}
```

## Implementation

**Create Cursor hook script:**

```python
#!/usr/bin/env python3
"""
Cursor session hook - Log Cursor Pro usage to session-log.jsonl

Triggered by Cursor on:
- Agent completion
- Composer stop
- Task finish
"""

import json
import os
from datetime import datetime
from pathlib import Path

SESSION_LOG = Path.home() / ".Codex" / "session-log.jsonl"


def log_cursor_task(task_type: str, prompt: str, model: str, tokens: int = 0):
    """Log Cursor task to session log."""
    
    SESSION_LOG.parent.mkdir(parents=True, exist_ok=True)
    
    # Read or create session
    session_id = os.environ.get("CURSOR_SESSION_ID")
    if not session_id:
        session_id = datetime.now().isoformat()
    
    # Create session entry
    session_entry = {
        "session_id": session_id,
        "start_time": datetime.now().isoformat(),
        "tasks": [{
            "task_type": task_type,
            "provider": "cursor-pro",  # Fixed for Cursor
            "tokens": tokens,
            "cost_usd": tokens * 0.00004 if tokens else 0,  # Rough estimate
            "model": model,
            "timestamp": datetime.now().isoformat()
        }],
        "total_tokens": tokens,
        "provider_usage": {
            "cursor-pro": {
                "tokens": tokens,
                "cost_usd": tokens * 0.00004 if tokens else 0,
                "task_count": 1
            }
        }
    }
    
    # Append to JSONL
    with open(SESSION_LOG, 'a') as f:
        f.write(json.dumps(session_entry) + '\n')


if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 3:
        print("Usage: cursor-session-hook.py <task_type> <model> [tokens]")
        sys.exit(1)
    
    task_type = sys.argv[1]
    model = sys.argv[2]
    tokens = int(sys.argv[3]) if len(sys.argv) > 3 else 0
    
    log_cursor_task(task_type, "cursor task", model, tokens)
```

**Cursor hook configuration:**

```json
// ~/.cursor/hooks.json
{
  "onAgentComplete": "~/.Codex/hooks/cursor-session-hook.py editor_native_coding {model} {tokens}",
  "onComposerStop": "~/.Codex/hooks/cursor-session-hook.py editor_native_coding {model} {tokens}"
}
```

**Alternative: Shell script wrapper**

```bash
#!/bin/bash
# ~/.Codex/hooks/cursor-hook.sh

TASK_TYPE="$1"
MODEL="$2"
TOKENS="$3"

SESSION_LOG="$HOME/.Codex/session-log.jsonl"

# Simple session entry
cat >> "$SESSION_LOG" <<EOF
{
  "session_id": "$(date -u +%Y-%m-%dT%H:%M:%S.%N)",
  "start_time": "$(date -u +%Y-%m-%dT%H:%M:%S.%N)",
  "tasks": [
    {
      "task_type": "$TASK_TYPE",
      "provider": "cursor-pro",
      "tokens": ${TOKENS:-0},
      "cost_usd": $(echo "scale=6; ${TOKENS:-0} * 0.00004" | bc),
      "model": "$MODEL",
      "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%S.%N)"
    }
  ],
  "total_tokens": ${TOKENS:-0},
  "provider_usage": {
    "cursor-pro": {
      "tokens": ${TOKENS:-0},
      "cost_usd": $(echo "scale=6; ${TOKENS:-0} * 0.00004" | bc),
      "task_count": 1
    }
  }
}
EOF
```

## Why This Matters

**Manual tracking (current):**
- Run `session-tracker.py analyze 30` after sessions
- Estimate tokens from prompt text
- No live visibility into Cursor quota burn-down

**Hook-based tracking (automated):**
- Every Cursor Agent completion logs automatically
- Real-time session data in `session-log.jsonl`
- `break-even-tracker.py` has live data for prepaid analysis

**Result:** Accurate prepaid utilization metrics without manual effort.

## Verification

```bash
# After Cursor task completes, check log
tail -1 ~/.Codex/session-log.jsonl | python3 -m json.tool

# Should show cursor-pro entry with timestamp
```

## See Also

- `session-tracker.py` — Session tracking utilities
- `break-even-tracker.py` — Prepaid utilization analysis
- `shadow-provider-registry.json` — Provider ID registry
