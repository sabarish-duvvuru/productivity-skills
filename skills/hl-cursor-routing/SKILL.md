---
name: Headless Cursor Routing
description: Use when applying cursor-first task routing heuristics for prepaid credit maximization.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-cursor-routing
  icon_style: craft-category-glyph-v1
---

# hl-cursor-routing

**Pattern**: Cursor-first task routing for prepaid credit maximization

## When to Use

- Routing editor-native tasks (small/medium context, under 50K tokens)
- Defaulting IDE work to Cursor Pro before other subscriptions
- Optimizing prepaid credit utilization

## Pattern

**Default routing heuristic:**

For small/medium context IDE work without explicit web/vision/privacy signals:
- **Context < 20K tokens** → Default to `editor_native_coding` (Cursor Pro)
- **Context 20-50K tokens** → Check for explicit signals first
  - "implement", "build", "create" → `implementation_coding` (ChatGPT/Codex)
  - "review", "analyze", "check" → `editor_native_coding` (Cursor Pro)
  - Otherwise → `editor_native_coding` (Cursor Pro)
- **Context > 50K tokens** → `large_context_distill` (Gemini bypass)

**Signal keywords that override default:**
- Vision → `vision_task` (GLM Global Coding Plan)
- Web/search → `web_search` (Search bypass)
- Private/sensitive → `private_local` (local Ollama)

## Implementation

```python
def classify_task_for_cursor(prompt_text: str, context_size: int) -> str:
    """Classify task with Cursor-first default."""

    # Explicit signal checks first
    prompt_lower = prompt_text.lower()

    if any(word in prompt_lower for word in ["search", "find", "look up", "google"]):
        return "web_search"

    if "private" in prompt_lower or "sensitive" in prompt_lower:
        return "private_local"

    if context_size > 100000:
        return "large_context_distill"

    # Cursor-first defaults for IDE work
    if context_size < 20000:
        return "editor_native_coding"  # Cursor Pro

    # Medium context: check intent
    if any(word in prompt_lower for word in ["build", "create", "implement", "add feature"]):
        return "implementation_coding"  # ChatGPT/Codex

    if any(word in prompt_lower for word in ["review", "analyze", "check", "debug", "fix"]):
        return "editor_native_coding"  # Cursor Pro

    # Default: Cursor Pro for IDE work
    return "editor_native_coding"
```

## Why This Matters

Prepaid credits (Cursor Pro, GLM Coding Plan) are sunk costs already paid. The goal is to extract value by using them before quotas reset.

Defaulting IDE work to Cursor Pro ensures:
- Fast request quota gets used (resets monthly)
- Prepaid value extracted, not wasted
- Correct provider attribution in session logs

## See Also

- `break-even-tracker.py` — Prepaid utilization analysis
- `task-router.py` — Optimal subscription routing
- `session-tracker.py` — Task classification and logging
