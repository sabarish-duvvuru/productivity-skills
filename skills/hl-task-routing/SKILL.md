---
name: Headless Task Routing
description: Use when routing tasks to the best subscription or provider based on task type, context size, and privacy.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-task-routing
  icon_style: craft-category-glyph-v1
---

# Task Routing

**Trigger**: "Which subscription to use", "best provider for this task", subscription selection

**Purpose**: Route tasks to optimal subscriptions based on task type, context size, privacy level.

## Core Pattern

Different tasks need different providers. No single subscription is optimal for everything.

## Atomic Routing Function

```python
def route_task(task_type, context_size=0, privacy_level="public"):
    """Route task to optimal subscription."""

    routing_matrix = {
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
        },
        "vision_task": {
            "primary": "glm-global-coding-plan",
            "fallback": "shadow-gemini-docs-bypass",
            "temp": 0.2,
            "cost_tier": "medium",
            "context": 32000
        },
        "large_context_distill": {
            "primary": "shadow-gemini-docs-bypass",
            "fallback": "glm-global-coding-plan",
            "temp": 0.0,
            "cost_tier": "specialized",
            "context": 200000
        },
        "web_search": {
            "primary": "shadow-search-bypass",
            "fallback": "glm-global-coding-plan",
            "temp": 0.7,
            "cost_tier": "specialized",
            "context": 16000
        },
        "private_local": {
            "primary": "local-shadow-ollama",
            "fallback": None,
            "temp": 0.1,
            "cost_tier": "local",
            "context": 8000
        }
    }

    # Privacy override
    if privacy_level == "private":
        return routing_matrix["private_local"]

    # Context override
    if context_size > 200000:
        return routing_matrix["large_context_distill"]

    # Get routing rule
    rule = routing_matrix.get(task_type, routing_matrix["implementation_coding"])
    return rule
```

## Prompt Detection

```python
def suggest_subscription_for_prompt(prompt_text, context_size=0):
    """Analyze prompt and suggest optimal subscription."""

    prompt_lower = prompt_text.lower()

    # Detect task type from prompt
    if "vision" in prompt_lower or "see" in prompt_lower:
        task_type = "vision_task"
    elif "review" in prompt_lower and "deep" in prompt_lower:
        task_type = "deep_review"
    elif "search" in prompt_lower or "find" in prompt_lower:
        task_type = "web_search"
    elif "private" in prompt_lower or "sensitive" in prompt_lower:
        task_type = "private_local"
    elif context_size > 100000:
        task_type = "large_context_distill"
    else:
        task_type = "implementation_coding"

    return route_task(task_type, context_size)
```

## Privacy-Tier Routing (GLM-001 Policy)

Before ANY routing decision, classify the data tier:

| Tier | Markers | GLM Allowed? | Routed To |
|------|---------|-------------|----------|
| **RED** | Astemo, Honda, FH4S, BOM, drawings, SASG, DFMEA, customer data, financials, trade secrets | ❌ **NEVER** | Local Ollama ONLY (or wait for Codex cap reset) |
| **AMBER** | Shadow Lab IP, patent drafts, mesh topology, personal financial, family, internal infra | ⚠️ Conditional | Gemini (US-hosted) preferred; GLM only with metadata stripped |
| **GREEN** | OSS code, public docs, web content, UI prototypes, non-sensitive coding, research papers | ✅ **Preferred** | GLM primary — maximize subscription value |

### Tier Detection

```python
def classify_tier(prompt_text, file_paths=None):
    """Classify data tier from prompt content and file context."""
    red_markers = ["astemo", "fh4s", "honda", "4e0a", "ipm", "sasg",
                   "dfmea", "drbfm", "change point", "bom", "drawing",
                   "mlc", "usgrp1", "greenfield", "tarboro", "keihin",
                   "hoe-axle", "smart drawing"]
    amber_markers = ["patent", "sp-00", "shadow-lab", "mesh topology",
                     "api key", "secret", "financial", "family",
                     "tailscale ip", "shadow-db"]

    text_lower = prompt_text.lower()
    paths_lower = " ".join(file_paths or []).lower()
    combined = text_lower + " " + paths_lower

    if any(m in combined for m in red_markers):
        return "RED"
    if any(m in combined for m in amber_markers):
        return "AMBER"
    return "GREEN"
```

### GLM Zone Gate

```python
def glm_gate(data_tier, zone):
    """Returns True if GLM routing is permitted."""
    if data_tier == "RED":
        return False  # Never, no exceptions
    if data_tier == "AMBER" and zone == "enterprise":
        return False  # Enterprise amber also blocked
    if data_tier == "AMBER":
        return True   # Allowed but prefer Gemini first
    return True       # GREEN — GLM preferred
```

### Patched Route Function

```python
def route_task(task_type, context_size=0, privacy_level="public", data_tier="GREEN", zone="personal"):
    rule = routing_matrix.get(task_type, routing_matrix["implementation_coding"])

    # GLM privacy gate — swap GLM fallback to local/Gemini for non-GREEN tiers
    if not glm_gate(data_tier, zone):
        if rule["primary"] == "glm-global-coding-plan":
            rule = {**rule, "primary": "local-shadow-ollama"}
        if rule.get("fallback") == "glm-global-coding-plan":
            rule = {**rule, "fallback": "shadow-gemini-docs-bypass"}

    # Existing privacy override (backward compat)
    if privacy_level == "private":
        return routing_matrix["private_local"]
    if context_size > 200000:
        return routing_matrix["large_context_distill"]

    return rule
```

## GLM-Preferred Lanes (GREEN Tier)

These task types should **prefer GLM** as primary to maximize subscription value:

| Task Type | Why GLM Primary | Volume Signal |
|-----------|----------------|---------------|
| `vision_task` | GLM-5.1 has strong vision; screenshots are non-sensitive | High |
| `implementation_coding` (public/OSS) | Max tokens on cheapest plan | Highest |
| `web_search` synthesis | Public data, high volume | Medium |
| Skill generation (GLI) | Output is public artifact | Medium |
| HTML prototype / UI | No confidential data in generated HTML | Medium |
| Non-sensitive research | Public information only | Medium |

## Rules

- **Never comfort-route** — always use optimal subscription for task type
- **Privacy first** — RED tier data NEVER reaches GLM, no exceptions
- **GLM-001 policy** — tier-gate before every routing decision
- **Context budget optimization** — don't waste long context on short tasks
- **Temperature matters** — different tasks need different creativity levels
- **Check fallbacks** — primary may be at budget limit
- **Enterprise zone = GLM hard block** — even if data_tier is GREEN, enterprise zone context may carry RED residue from earlier prompts

**Crystallizes from**: shadow-task-routing-matrix.json 2026-04-02 - task routing patterns + GLM-001 privacy audit 2026-04-30.

**Atomic output**: Recommended subscription + fallback + temperature + context limit + tier classification.
