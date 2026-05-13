---
name: 'Hook Performance'
description: Use when auditing and optimize Codex hook performance — benchmark startup and per-call hooks, find redundant observers, measure prompt injection overhead, recommend fixes. Trigger on "hook performance", "hook audit", "slow hooks", "optimize hooks", "hook overhead", "benchmark hooks", "why is startup slow", "per-call latency".
icon: icon.svg
metadata:
  category: system/performance
  family: workflow
  lifecycle: active
  canonical_slug: hook-perf
  icon_style: craft-category-glyph-v1
---

# hook-perf — Hook Performance Auditor

Deep-dive into Codex hook latency, redundancy, and prompt injection overhead.

## What This Measures

| Layer | What | Why it matters |
|-------|------|----------------|
| **SessionStart** | Banner + init scripts | Delay before first response |
| **PreToolUse** | Prompt hooks per tool call | Context bloat on every action |
| **PostToolUse** | Command hooks per tool call | Cumulative latency (N calls x M ms) |
| **UserPromptSubmit** | Inbox checks, dispatchers | Latency on every user message |
| **Stop/Compact** | Anchor saves | End-of-session overhead |

## Step 1: Collect hook configuration

Read both settings files to build the hook inventory:

```bash
echo "=== PROJECT SETTINGS ==="
cat "$(pwd)/.Codex/settings.json" 2>/dev/null || echo "(none)"

echo "=== PROJECT LOCAL SETTINGS ==="
cat "$(pwd)/.Codex/settings.local.json" 2>/dev/null || echo "(none)"

echo "=== GLOBAL SETTINGS ==="
cat ~/.Codex/settings.json 2>/dev/null || echo "(none)"
```

Parse every hook entry. For each, record:
- **Event** (SessionStart, PreToolUse, PostToolUse, etc.)
- **Type** (command or prompt)
- **Matcher** (which tools it fires on — empty = ALL tools)
- **Timeout** setting
- **Script path** (for command hooks)

## Step 2: Benchmark command hooks

For every command hook found, run it with empty/minimal stdin and measure wall time:

```bash
# Template — adapt for each hook script found
echo '{}' | timeout 5 /usr/bin/time -p python3 <script_path> 2>&1
```

Record wall time for each. Flag any hook that:
- Takes >50ms (cumulative impact on per-call hooks)
- Takes >200ms (startup impact)
- Produces deprecation warnings
- Fails or returns non-zero

## Step 3: Analyze prompt hooks

For each prompt-type hook, assess:

1. **Matcher breadth**: Does it fire on a specific tool (`Edit`) or broadly (`Bash`, empty)?
2. **Content size**: How many tokens does the prompt inject?
3. **Relevance rate**: What % of matched tool calls actually need this guidance?

Flag prompt hooks where:
- Matcher is empty (fires on ALL tool calls)
- Matcher is `Bash` without narrowing (fires on `ls`, `git status`, etc.)
- Prompt content exceeds ~100 tokens
- Guidance is already covered in AGENTS.md rules

## Step 4: Detect redundancy

Cross-reference project vs global hooks. Look for:
- **Duplicate signal collection**: Two hooks logging the same tool calls
- **Overlapping prompt hooks**: Same guidance injected at multiple layers
- **Rules already in AGENTS.md**: Prompt hooks repeating what's in rules/ files

## Step 5: Calculate overhead

Compute per-session cost estimate:

```
Per tool call overhead = sum(command hook times) + count(prompt hooks) * ~50 tokens
Estimated calls/session = 150 (typical)
Session overhead = per_call * 150 + startup_time
```

## Output: Performance Report

```
Hook Performance Report
━━━━━━━━━━━━━━━━━━━━━━━━━━

Startup Latency
  astemo-banner.py          22ms  ✓
  shadow-connect-init.py    62ms  ✓
  config-evolution.py       38ms  ✓
  build-registry.py         39ms  (async)
  Total startup:           ~122ms

Per-Call Overhead (PostToolUse)
  post-tool-dispatch.py     30ms  (global, every call)
  tool_observer collect     38ms  (project, every call)  ⚠ redundant?
  Edit|Write prompt         ~50 tokens
  Total per call:          ~68ms + 50 tokens

Per-Call Overhead (PreToolUse)
  Bash prompt               ~80 tokens on every Bash call  ⚠ broad matcher
  Edit prompt               ~60 tokens

Session Estimate (200 calls)
  Command hooks:           ~13.6s total
  Prompt injection:        ~26K tokens total
  Startup:                 ~122ms

Recommendations
  1. [HIGH] Narrow Bash PreToolUse matcher → Bash(python3*)
  2. [MED]  Remove tool_observer if post-tool-dispatch covers it
  3. [LOW]  Fix datetime.utcnow() deprecation in tool_observer
```

## Stop Condition

After presenting the report, ask:
> "Want me to apply the fixes? I'll handle each one separately with before/after benchmarks."

Do not modify any settings or scripts without explicit confirmation.
