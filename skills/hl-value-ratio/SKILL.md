---
name: Headless Value Ratio
description: Use when calculating subscription value ratios from equivalent API cost versus actual spend.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-value-ratio
  icon_style: craft-category-glyph-v1
---

# Value Ratio Calculator

**Trigger**: Subscription analysis, "is this subscription worth it", cost optimization

**Purpose**: Calculate value ratio (equivalent API cost vs actual spend) for any subscription.

## Core Pattern

Value ratio = (equivalent API cost) / (actual subscription cost)

- **ratio ≥ 1.0**: HEALTHY — getting more value than cost
- **ratio < 1.0**: UNHEALTHY — paying more than equivalent API cost

## Atomic Function

```python
def calculate_value_ratio(subscription_id, period="monthly"):
    """Calculate value ratio for subscription."""

    subscriptions = {
        "cursor-pro": {
            "monthly_cost": 20,
            "fast_included": 500,
            "api_equivalent_cost_per_1m": 2.50  # GPT-4 Turbo
        },
        "glm-global-coding-plan": {
            "monthly_cost": 0,
            "prompt_cap_5h": 2400,
            "api_equivalent_cost_per_1m": 0.15
        },
        "chatgpt-pro-200": {
            "monthly_cost": 20,
            "api_equivalent_cost_per_1m": 15.00  # o4-mini or o3
        }
    }

    sub = subscriptions[subscription_id]
    actual_spend = sub["monthly_cost"]
    actual_tokens = get_usage_tokens(subscription_id)  # from usage registry

    # Calculate equivalent API cost
    equivalent_cost = actual_tokens * sub["api_equivalent_cost_per_1m"]

    # Value ratio
    if actual_spend > 0:
        value_ratio = equivalent_cost / actual_spend
    else:
        value_ratio = float('inf')  # Free subscription

    return {
        "subscription": subscription_id,
        "value_ratio": round(value_ratio, 2),
        "status": "HEALTHY" if value_ratio >= 1.0 else "UNHEALTHY",
        "actual_spend_usd": actual_spend,
        "equivalent_api_cost_usd": round(equivalent_cost, 2)
    }
```

## Break-Even Analysis

```python
def break_even_analysis(subscription_id, task_patterns):
    """Calculate break-even point for subscription."""

    # Example: cursor-pro
    # Cost: $20/month
    # Value: 15 min saved per session × $100/hour = $25/session
    # Break-even: 3 sessions/week × $25 = $75/week > $20/month

    sessions_per_week = len([t for t in task_patterns if t["type"] == "editor"])
    weekly_value = sessions_per_week * 15 / 60 * 100  # hours × rate
    value_ratio = weekly_value / 20  # subscription cost

    return {
        "break_even_status": "MET" if value_ratio >= 1.0 else "NOT_MET",
        "sessions_per_week": sessions_per_week,
        "weekly_value_usd": weekly_value
    }
```

## Rules

- **Track value ratios monthly** — subscriptions drift over time
- **Pause unhealthy subscriptions** — ratio < 1.0 for 2+ months
- **Consider equivalent API cost** — what would this cost à la carte?
- **Factor in time savings** — operator time has monetary value
- **GLM-001**: Value ratio calculation for GLM only counts GREEN-tier traffic. RED/enterprise traffic MUST NOT inflate GLM value metrics — it should never reach GLM in the first place.

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 - value ratio calculations.

**Atomic output**: Value ratio + health status + break-even analysis.


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

