---
name: Headless Subscription Maximize
description: 'Use when the user needs: Maximize subscription value by routing work to the right prepaid or recurring provider lanes.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-subscription-maximize
  icon_style: craft-category-glyph-v1
---

# Subscription Maximization

**Trigger**: Cost analysis, "which subscription to use", "over budget"

**Purpose**: Maximize subscription value by routing tasks to optimal providers.

## Core Pattern

**Philosophy**: Each subscription used only where true advantage. Avoid comfort-routing. Broker routes only broker-ready lanes.

## Atomic Subscription Analysis

### 1. Cost Tier Classification

```python
def analyze_cost_per_task(subscription, task_type, session_count):
    """Determine if subscription provides value."""
    subscriptions = {
        "cursor-pro": {
            "monthly_usd": 20,
            "fast_requests": 500,
            "break_even_sessions_per_week": 3,
            "cost_per_fast_req": 0.04
        },
        "glm-global-coding-plan": {
            "monthly_usd": 0,
            "prompt_cap_5h": 2400,
            "cost_per_prompt": 0.001,
            "value_ratio_min": 1.0
        },
        "chatgpt-pro-200": {
            "monthly_usd": 0,
            "operator_seat": True,
            "api_equivalent": "o4-mini or o3",
            "break_even_sessions": 5
        }
    }
    
    sub = subscriptions[subscription]
    current_usage = session_count / 4  # weekly sessions
    
    # Calculate break-even
    if subscription == "cursor-pro":
        break_even = sub["break_even_sessions_per_week"]
        value_ratio = current_usage / break_even
        return {
            "subscription": subscription,
            "value_ratio": value_ratio,
            "recommendation": "KEEP" if value_ratio >= 1.0 else "DOWNGRADE/Pause"
        }
```

### 2. Task-to-Subscription Mapping

```python
task_routing = {
    "deep_review": {
        "primary": "cursor-pro",
        "fallback": "glm-global-coding-plan",
        "temp": 0.3,
        "cost_tier": "high",
        "context": 80000
    },
    "implementation_coding": {
        "primary": "chatgpt-pro-200",
        "fallback": "glm-global-coding-plan",
        "temp": 0.1,
        "cost_tier": "medium",
        "context": 20000
    },
    "editor_native_coding": {
        "primary": "cursor-pro",
        "fallback": "chatgpt-pro-200",
        "temp": 0.05,
        "cost_tier": "low",
        "context": 4000
    }
}
```

### 3. Value Ratio Calculator

```python
def calculate_value_ratio(subscription, period="monthly"):
    """Calculate subscription value ratio (actual cost / amortized value)."""
    
    # Track actual usage
    actual_tokens = get_usage_stats(subscription, period)
    actual_spend = get_spend(subscription, period)
    
    # Calculate equivalent API cost
    api_price_per_1k = get_api_pricing(subscription)
    equivalent_cost = (actual_tokens / 1000) * api_price_per_1k
    
    # Value ratio
    if actual_spend > 0:
        value_ratio = equivalent_cost / actual_spend
    else:
        value_ratio = float('inf')  # Free subscription
    
    return {
        "subscription": subscription,
        "period": period,
        "value_ratio": value_ratio,
        "status": "HEALTHY" if value_ratio >= 1.0 else "UNHEALTHY",
        "recommendation": optimize_subscription(value_ratio, subscription)
    }
```

### 4. Break-Even Analysis

```python
def break_even_analysis(subscription, task_patterns):
    """Calculate break-even points for subscription value."""
    
    cursor_pro_example = {
        "monthly_cost": 20,
        "fast_included": 500,
        "cost_per_fast": 0.04,
        "sessions_needed": 3
    }
    
    # Calculate
    sessions_per_week = len([p for p in task_patterns if p["type"] == "editor"])
    time_saved_per_session = 15  # minutes
    
    weekly_value = sessions_per_week * time_saved_per_session / 60  # hours
    hourly_rate = 100  # $/hour (operator rate)
    weekly_value_usd = weekly_value * hourly_rate
    
    value_ratio = weekly_value_usd / 20  # cursor-pro cost
    
    return {
        "subscription": "cursor-pro",
        "break_even_status": "MET" if value_ratio >= 1.0 else "NOT_MET",
        "current_value_ratio": value_ratio,
        "optimization": "Increase usage to 3+ sessions/week or pause"
    }
```

## Atomic Agent Functions

### Task Router (GLM-001 Policy)

```python
# Data tier markers — shared with hl-task-routing
def classify_tier(prompt_text, file_paths=None):
    red_markers = ["astemo", "fh4s", "honda", "4e0a", "ipm", "sasg",
                   "dfmea", "drbfm", "change point", "bom", "drawing",
                   "mlc", "usgrp1", "greenfield", "tarboro"]
    amber_markers = ["patent", "sp-00", "shadow-lab", "mesh topology",
                     "api key", "secret", "financial", "family"]
    combined = (prompt_text + " " + " ".join(file_paths or [])).lower()
    if any(m in combined for m in red_markers): return "RED"
    if any(m in combined for m in amber_markers): return "AMBER"
    return "GREEN"

def glm_gate(data_tier, zone):
    """True if GLM routing is permitted under GLM-001."""
    if data_tier == "RED": return False
    if data_tier == "AMBER" and zone == "enterprise": return False
    return True

def route_task_to_subscription(task_type, context_size, privacy_level,
                                data_tier=None, zone="personal"):
    """Route task to optimal subscription with GLM-001 privacy gate."""
    if data_tier is None:
        data_tier = "RED" if privacy_level == "private" else "GREEN"

    # Hard privacy override
    if privacy_level == "private" or data_tier == "RED":
        return "local-shadow-ollama"

    # Context override
    if context_size > 200000:
        return "shadow-gemini-docs-bypass"

    # GLM-preferred lanes (GREEN tier only)
    glm_primary_lanes = {
        "vision_task": "glm-global-coding-plan",
        "burst_overflow": "glm-global-coding-plan",
    }

    glm_allowed = glm_gate(data_tier, zone)

    rules = {
        "editor_task_small_context": "cursor-pro",
        "editor_task_large_context": "chatgpt-pro-200",
        "vision_task": "glm-global-coding-plan" if glm_allowed else "shadow-gemini-docs-bypass",
        "large_context_distill": "shadow-gemini-docs-bypass",
        "web_search": "shadow-search-bypass",
        "burst_overflow": "glm-global-coding-plan" if glm_allowed else "shadow-gemini-docs-bypass",
    }

    # Default: GLM for GREEN, Gemini for AMBER
    return rules.get(task_type, "glm-global-coding-plan" if glm_allowed else "shadow-gemini-docs-bypass")
```

### Cost Tracker

```python
def track_subscription_usage(subscription, task_tokens, cost):
    """Track usage and alert on budget thresholds."""
    
    thresholds = {
        "cursor-pro": {"monthly_budget_usd": 20, "alert_pct": 0.80},
        "glm-global-coding-plan": {"prompt_cap_5h": 2400, "alert_pct": 0.90},
        "openrouter": {"monthly_budget_usd": 15, "alert_pct": 0.67}
    }
    
    threshold = thresholds[subscription]
    current_usage = get_monthly_usage(subscription)
    
    if "monthly_budget_usd" in threshold:
        usage_pct = current_usage / threshold["monthly_budget_usd"]
    elif "prompt_cap_5h" in threshold:
        usage_pct = current_usage / threshold["prompt_cap_5h"]
    
    if usage_pct >= threshold["alert_pct"]:
        return {
            "subscription": subscription,
            "alert": "BUDGET_ALERT",
            "usage_pct": usage_pct,
            "action": "Review routing or pause subscription"
        }
```

## GLM Value Maximization Targets

| Metric | Current | Target |
|--------|---------|--------|
| Monthly sessions | 41 | 50-60 |
| Monthly tokens | 8.6M | 10-12M |
| GREEN-tier share | ~75% | 90%+ |
| RED-tier leakage | ~20% (estimated) | 0% |
| Value ratio | ∞ (free plan) | Maintain ∞ |

## Rules

- **GLM-001 policy** — RED tier NEVER reaches GLM; AMBER conditional
- **Never comfort-route** — always use optimal subscription for task type
- **Track value ratios** — subscription cost vs equivalent API cost
- **Monitor break-even** — pause subscriptions not meeting minimum usage
- **Broker-routable lanes** for burst overflow, not operator seats
- **Private data stays local** — no exceptions
- **Temperature matters** — different tasks need different creativity levels
- **Context budget optimization** — don't waste long context on short tasks
- **Maximize GREEN lanes** — route public coding, vision, web synthesis to GLM first

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 + GLM-001 privacy audit 2026-04-30.

**Atomic output**: Cost analysis + routing recommendations + break-even calculations + tier classification.
