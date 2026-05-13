---
name: Headless Break Even
description: Use when calculating break-even points for subscriptions from actual time savings and delivered value.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-break-even
  icon_style: craft-category-glyph-v1
---

# Break-Even Analysis

**Trigger**: Subscription evaluation, "is this subscription worth it", minimum usage calculation

**Purpose**: Calculate break-even points for subscriptions based on actual value delivered.

## Core Pattern

Break-even = (time saved × hourly rate) / subscription cost

If ratio ≥ 1.0, subscription pays for itself in time value.

## Atomic Break-Even Function

```python
def break_even_analysis(subscription_id, task_patterns):
    """Calculate break-even point for subscription."""

    # Example: cursor-pro
    # Cost: $20/month
    # Sessions per week needed: 3
    # Time saved per session: 15 minutes
    # Hourly operator rate: $100/hour

    subscriptions = {
        "cursor-pro": {
            "monthly_cost": 20,
            "fast_included": 500,
            "cost_per_fast": 0.04,
            "sessions_needed_weekly": 3,
            "time_saved_per_session_min": 15
        },
        "chatgpt-pro-200": {
            "monthly_cost": 20,
            "sessions_needed_weekly": 5,
            "time_saved_per_session_min": 20
        }
    }

    if subscription_id not in subscriptions:
        return {"error": f"Unknown subscription: {subscription_id}"}

    sub = subscriptions[subscription_id]

    # Count actual sessions from task patterns
    sessions_per_week = len([t for t in task_patterns if t.get("type") == "editor"])

    # Calculate value
    time_saved_per_session = sub["time_saved_per_session_min"]
    weekly_value_hours = sessions_per_week * time_saved_per_session / 60
    hourly_rate = 100  # $/hour (operator rate)
    weekly_value_usd = weekly_value_hours * hourly_rate

    # Value ratio
    value_ratio = weekly_value_usd / sub["monthly_cost"]

    return {
        "subscription": subscription_id,
        "break_even_status": "MET" if value_ratio >= 1.0 else "NOT_MET",
        "current_value_ratio": round(value_ratio, 2),
        "sessions_per_week": sessions_per_week,
        "sessions_needed": sub["sessions_needed_weekly"],
        "weekly_value_usd": round(weekly_value_usd, 2),
        "recommendation": (
            f"Increase usage to {sub['sessions_needed_weekly']}+ sessions/week or pause"
            if value_ratio < 1.0 else "KEEP - meeting break-even"
        )
    }
```

## Session-Based Calculation

```python
def break_even_from_sessions(subscription_id, sessions_per_week):
    """Calculate break-even from session count alone."""

    # cursor-pro example
    monthly_cost = 20
    time_saved_min = 15  # per session
    hourly_rate = 100

    # Weekly value
    weekly_time_saved_hours = sessions_per_week * time_saved_min / 60
    weekly_value_usd = weekly_time_saved_hours * hourly_rate

    # Monthly value (4.33 weeks per month)
    monthly_value_usd = weekly_value_usd * 4.33

    # Break-even ratio
    value_ratio = monthly_value_usd / monthly_cost

    return {
        "sessions_per_week": sessions_per_week,
        "monthly_value_usd": round(monthly_value_usd, 2),
        "subscription_cost_usd": monthly_cost,
        "value_ratio": round(value_ratio, 2),
        "break_even": value_ratio >= 1.0
    }
```

## Rules

- **Track actual sessions** — use task patterns or editor logs
- **Value your time** — operator rate = $100/hour (adjust as needed)
- **Time saved matters** — what would this take manually?
- **Pause if NOT_MET** — 2+ months below break-even

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 - break-even calculations.

**Atomic output**: Break-even status + value ratio + session recommendation.


## Pipeline

```
Intent → Resolve target → Execute → Validate → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`

## Contract

- Destructive operations require explicit operator confirmation
- Always dry-run before applying changes

