---
name: Headless Cost Tracking
description: 'Use when the user needs: Track subscription usage and budget thresholds for prepaid and recurring AI tooling.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-cost-tracking
  icon_style: craft-category-glyph-v1
---

# Cost Tracking

**Trigger**: Subscription budget monitoring, "over budget alerts", usage tracking

**Purpose**: Track subscription usage and alert when approaching budget thresholds.

## Core Pattern

Monitor usage percentage and alert at threshold (67%, 75%, 80%, 90%).

## Atomic Tracking Function

```python
def track_usage(subscription_id, task_tokens, cost_usd=0):
    """Track usage for subscription and update totals."""

    usage_registry = Path.home() / ".Codex" / "subscription-usage.json"

    # Load existing usage
    if usage_registry.exists():
        with open(usage_registry) as f:
            usage_data = json.load(f)
    else:
        usage_data = {}

    # Initialize subscription entry
    if subscription_id not in usage_data:
        usage_data[subscription_id] = {
            "total_tokens": 0,
            "total_cost_usd": 0,
            "task_count": 0,
            "data_tier_breakdown": {"GREEN": 0, "AMBER": 0, "RED": 0}  # GLM-001 tracking
        }

    # Update totals
    entry = usage_data[subscription_id]
    entry["total_tokens"] += task_tokens
    entry["total_cost_usd"] += cost_usd
    entry["task_count"] += 1
    entry["last_updated"] = datetime.now().isoformat()

    # Save
    usage_registry.parent.mkdir(parents=True, exist_ok=True)
    with open(usage_registry, 'w') as f:
        json.dump(usage_data, f, indent=2)

    return entry
```

## Threshold Checking

```python
def check_thresholds(subscription_id):
    """Check if subscription is near budget threshold."""

    thresholds = {
        "cursor-pro": {"monthly_budget_usd": 20, "alert_pct": 0.80},
        "glm-global-coding-plan": {"prompt_cap_5h": 2400, "alert_pct": 0.90},
        "chatgpt-pro-200": {"monthly_budget_usd": 20, "alert_pct": 0.75},
        "openrouter": {"monthly_budget_usd": 15, "alert_pct": 0.67}
    }

    if subscription_id not in thresholds:
        return {"error": f"No threshold defined for {subscription_id}"}

    threshold = thresholds[subscription_id]
    current_usage = load_usage(subscription_id)

    # Calculate usage percentage
    if "monthly_budget_usd" in threshold:
        usage_pct = current_usage["total_cost_usd"] / threshold["monthly_budget_usd"]
        usage_type = "USD"
    elif "prompt_cap_5h" in threshold:
        usage_pct = current_usage["total_tokens"] / threshold["prompt_cap_5h"]
        usage_type = "prompts"
    else:
        return {"error": "Unknown threshold type"}

    # Check alert threshold
    if usage_pct >= threshold["alert_pct"]:
        return {
            "subscription": subscription_id,
            "alert": "BUDGET_ALERT",
            "usage_pct": round(usage_pct * 100, 1),
            "action": "Review routing or pause subscription"
        }

    return {
        "subscription": subscription_id,
        "status": "OK",
        "usage_pct": round(usage_pct * 100, 1)
    }
```

## Rules

- **Alert before overage** — 67%, 75%, 80%, 90% thresholds prevent surprise charges
- **Track per subscription** — separate registries for each provider
- **Reset monthly** — clear counters at billing cycle start
- **Log all usage** — tokens, cost, task count for trend analysis
- **GLM-001 tier tracking** — log data_tier for every GLM request; RED count must be 0
- **Enterprise leak alert** — if `data_tier_breakdown.RED > 0` for `glm-global-coding-plan`, trigger immediate CRITICAL alert

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 - budget threshold patterns.

**Atomic output**: Usage percentage + alert status + action recommendation.


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

