---
name: 'Headless Provider ID Normalization'
description: 'Use when the user needs: Normalize provider IDs across registries, session logs, and analyzers to prevent drift.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-provider-id-normalization
  icon_style: craft-category-glyph-v1
---

# hl-provider-id-normalization

**Pattern**: Normalize provider IDs across registry, session logs, and analyzers

## When to Use

- Fixing provider ID drift causing underreporting
- Ensuring session logs match prepaid utilization analysis
- Adding new subscriptions to tracking system

## Pattern

**Single source of truth for provider IDs:**

All systems MUST use identical provider IDs:
- Registry: `shadow-provider-registry.json`
- Task router: `task-router.py`
- Session tracker: `session-tracker.py`
- Prepaid analyzer: `break-even-tracker.py`
- Usage registry: `~/.Codex/subscription-usage.json`

**Canonical provider IDs:**

```json
{
  "cursor-pro": "Cursor Pro subscription",
  "chatgpt-pro-200": "ChatGPT Pro-200 / Codex Pro",
  "glm-global-coding-plan": "GLM Global Coding Plan (Z.AI prepaid)",
  "shadow-gemini-bypass": "Shadow Gemini Bypass Bridge",
  "shadow-search-bypass": "Shadow Search Bypass Bridge",
  "local-shadow": "Local SHADOW execution"
}
```

## Common Mismatches

**❌ WRONG:**
```python
# In session-tracker.py
provider = "glm-coding-plan"

# In break-even-tracker.py
analyze_prepaid_utilization("glm-coding-plan")

# In task-router.py
routing["glm-global-coding-plan"] = {...}
```

**✅ CORRECT:**
```python
# Everywhere
PROVIDer_ID = "glm-global-coding-plan"
```

## Implementation

**Step 1: Create provider ID constants**

```python
# provider_ids.py
CURSOR_PRO = "cursor-pro"
CHATGPT_PRO_200 = "chatgpt-pro-200"
GLM_GLOBAL_CODING_PLAN = "glm-global-coding-plan"
SHADOW_GEMINI_BYPASS = "shadow-gemini-bypass"
SHADOW_SEARCH_BYPASS = "shadow-search-bypass"
LOCAL_SHADOW = "local-shadow"
```

**Step 2: Import constants everywhere**

```python
# In task-router.py
from provider_ids import GLM_GLOBAL_CODING_PLAN, CURSOR_PRO

# In session-tracker.py
from provider_ids import GLM_GLOBAL_CODING_PLAN, CURSOR_PRO

# In break-even-tracker.py
from provider_ids import GLM_GLOBAL_CODING_PLAN, CURSOR_PRO
```

**Step 3: Update shadow-provider-registry.json**

Ensure `providerId` fields match constants:
- `$schema` ID field
- Agent system references
- Task routing matrix references

## Why This Matters

Session logs write provider ID → prepaid analyzer reads provider ID.

If IDs don't match:
- GLM utilization reads as 0 (wrong ID checked)
- Break-even analysis fails
- Recommendations are inaccurate

> **GLM-001**: The provider ID `glm-global-coding-plan` is subject to the GLM-001 privacy policy. RED-tier (enterprise/Astemo) data must NEVER be routed to this provider regardless of which system references the ID.

**Result:** You think you're not using prepaid credits when you actually are.

## Verification

```bash
# Check for provider ID drift
grep -r "glm-coding-plan" ~/.Codex/skills/
grep -r "glm-global-coding-plan" ~/.Codex/skills/

# Should return: 0 matches for glm-coding-plan
# Should return: matches for glm-global-coding-plan
```

## See Also

- `shadow-provider-registry.json` — Canonical provider ID registry
- `session-tracker.py` — Writes provider IDs to session logs
- `break-even-tracker.py` — Reads provider IDs for analysis
