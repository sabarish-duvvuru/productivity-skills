---
name: Headless Session Tracking
description: 'Use when the user needs: Track real Codex session usage, tokens, and costs for subscription value analysis.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-session-tracking
  icon_style: craft-category-glyph-v1
---

# hl-session-tracking

Track actual Codex/Codex session usage for accurate subscription value monitoring.

## Purpose

Real usage tracking beats estimates. Capture tokens, costs, and task patterns to calculate accurate value ratios.

## When to Use

- After completing a coding task to log actual usage
- Before making subscription decisions to see real value data
- Monthly to review cost-per-day trends and task distribution
- When verifying if subscriptions are worth their cost

## Usage

### Track Session

```python
from ShadowArchive.03_agent_roots.subscription_maximize.session_tracker import SessionTracker

tracker = SessionTracker()
tracker.log_task(
    task_type="deep_review",
    provider="cursor-pro",
    tokens=15000,
    cost_usd=0.25,
    context_size=50000
)
tracker.end_session()
```

### Analyze History

```bash
# Last 30 days
python3 session-tracker.py analyze 30

# Last 7 days via swarm
python3 swarm-orchestrator.py sessions 7

# Full report with history
python3 swarm-orchestrator.py report
```

## Data Storage

- **Session Log:** `~/.Codex/session-log.jsonl` — Append-only JSONL
- **Usage Registry:** `~/.Codex/subscription-usage.json` — Aggregate totals
- **Last Session:** `~/.Codex/last-session.json` — Quick reference

## Task Classification

Auto-detected from prompt text:
- `deep_review` — Thorough code reviews
- `implementation_coding` — Feature implementation
- `editor_native_coding` — Quick edits in IDE
- `vision_task` — Screenshot analysis
- `web_search` — Lookups and searches
- `private_local` — Sensitive data processing
- `large_context_distill` — Big context reduction

## Value Tracking

Accurate value ratios require real usage data:

```
value_ratio = equivalent_api_cost / actual_spend

Before session tracking:
  cursor-pro: ratio 0.0 (no usage data)

After 10 sessions tracked:
  cursor-pro: ratio 1.25 (getting $25 value for $20 cost)
```

## Hook Integration

Automatic tracking via `.Codex/hooks/session-tracker.py`:
- Runs on session end
- Classifies task from last prompt
- Estimates tokens and cost
- Updates usage registry

## Key Patterns

### Token Estimation

```python
def estimate_tokens(prompt_text: str) -> int:
    # Rough approximation: 1 token ≈ 4 characters
    char_count = len(prompt_text)
    estimated_tokens = char_count // 4
    
    # Add buffer for response (2-5x prompt length)
    return estimated_tokens * 3
```

### Cost Calculation

```python
# Cost per million tokens by provider
COST_PER_MILLION = {
    "glm-global-coding-plan": 0.5,
    "chatgpt-pro-200": 2.0,
    "cursor-pro": 1.5,
    "shadow-gemini-docs-bypass": 0.1
}

cost_usd = (tokens / 1_000_000) * COST_PER_MILLION[provider]
```

### Historical Analysis

```python
# Daily averages from session history
daily_avg_tasks = total_tasks / days
daily_avg_tokens = total_tokens / days
cost_per_day = total_cost_usd / days

# Task distribution
task_type_distribution = {
    "deep_review": 10,
    "implementation_coding": 30,
    "vision_task": 5
}
```

## GLM-001 Tier Policy Integration

When tracking GLM sessions, log the data tier to enable enterprise leak audits:

```python
# Extended task classification
def classify_with_tier(prompt_text, file_paths=None):
    """Auto-detect task type AND data tier for session logging."""
    task_types = {
        "deep_review": ["review", "audit", "analyze code"],
        "implementation_coding": ["implement", "build", "create feature"],
        "editor_native_coding": ["edit", "fix", "quick change"],
        "vision_task": ["screenshot", "image", "see this", "looks like"],
        "web_search": ["search", "find", "lookup"],
        "private_local": ["private", "sensitive", "confidential"],
        "large_context_distill": ["summarize", "distill", "compress"]
    }

    red_markers = ["astemo", "fh4s", "honda", "4e0a", "sasg", "dfmea", "drbfm",
                   "change point", "bom", "drawing", "mlc", "usgrp1"]
    amber_markers = ["patent", "sp-00", "shadow-lab", "mesh topology",
                     "api key", "secret", "financial"]

    combined = (prompt_text + " " + " ".join(file_paths or [])).lower()
    tier = "GREEN"
    if any(m in combined for m in red_markers): tier = "RED"
    elif any(m in combined for m in amber_markers): tier = "AMBER"

    # Detect task type
    task_type = "implementation_coding"
    for ttype, keywords in task_types.items():
        if any(k in combined for k in keywords):
            task_type = ttype
            break

    return task_type, tier
```

### Enterprise Leak Detection

```python
def check_glm_leaks(session_log_path="~/.Codex/session-log.jsonl"):
    """Audit GLM sessions for RED-tier data leakage."""
    leaks = []
    with open(session_log_path) as f:
        for line in f:
            entry = json.loads(line)
            if entry.get("provider") == "glm-global-coding-plan":
                if entry.get("data_tier") == "RED":
                    leaks.append(entry)

    if leaks:
        return {
            "alert": "CRITICAL",
            "message": f"{len(leaks)} RED-tier sessions routed to GLM — policy violation",
            "sessions": leaks
        }
    return {"status": "OK", "message": "No RED-tier GLM leakage detected"}
```

## Integration Points

- **SessionTracker:** `ShadowArchive/03-agent-roots/subscription-maximize/session-tracker.py`
- **Swarm Orchestrator:** `swarm-orchestrator.py sessions [days]`
- **Hook:** `.Codex/hooks/session-tracker.py`
- **Value Ratios:** `subscription-analyzer.py` uses tracked data for accurate calculations

## Rules

- Enable hook for automatic session-end tracking
- Review weekly for usage patterns and tier distribution
- **GLM-001**: Never log RED-tier sessions to GLM provider; if detected, flag CRITICAL
- **Tier audit**: Run `check_glm_leaks()` monthly to verify zero enterprise leakage

## Crystallization Source

Session tracking system built 2026-04-19, crystallized from subscription optimization needs + desire for accurate usage data vs estimates.


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

