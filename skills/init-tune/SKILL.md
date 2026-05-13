---
name: 'Initialization Tuning'
description: 'Use when the user needs: Quick startup diagnostic and tuning for Codex sessions — time each init layer, show context budget, identify bloat. Light-weight complement to hook-perf. Trigger on "init tune", "startup speed", "session startup", "why slow start", "context budget", "init diagnostic", "tune startup".'
icon: icon.svg
metadata:
  category: system/performance
  family: design
  lifecycle: active
  canonical_slug: init-tune
  icon_style: craft-category-glyph-v1
---

# init-tune — Startup Speed Diagnostic

Fast, light diagnostic for Codex session initialization. Shows where time and context tokens go.

## Run Sequence

### 1. Time each startup layer

Run all benchmarks in a single bash block:

```bash
echo "=== STARTUP LAYERS ==="

# Layer 1: SessionStart command hooks (project)
for f in $(python3 -c "
import json, sys
try:
    d = json.load(open('$(pwd)/.Codex/settings.json'))
    for entry in d.get('hooks',{}).get('SessionStart',[]):
        for h in entry.get('hooks',[]):
            if h.get('type')=='command': print(h['command'])
except: pass
"); do
    echo "--- $f ---"
    START=$(python3 -c "import time; print(time.perf_counter())")
    eval "$f" > /dev/null 2>&1
    END=$(python3 -c "import time; print(time.perf_counter())")
    python3 -c "print(f'{(${END}-${START})*1000:.0f}ms')"
done

# Layer 2: SessionStart command hooks (global)
echo "=== GLOBAL STARTUP ==="
for f in $(python3 -c "
import json, sys
try:
    d = json.load(open('$HOME/.Codex/settings.json'))
    for entry in d.get('hooks',{}).get('SessionStart',[]):
        for h in entry.get('hooks',[]):
            if h.get('type')=='command': print(h['command'])
except: pass
"); do
    echo "--- $f ---"
    START=$(python3 -c "import time; print(time.perf_counter())")
    eval "$f" > /dev/null 2>&1
    END=$(python3 -c "import time; print(time.perf_counter())")
    python3 -c "print(f'{(${END}-${START})*1000:.0f}ms')"
done
```

### 2. Count context budget at startup

Estimate tokens consumed before the first user message:

```bash
echo "=== CONTEXT BUDGET ==="
python3 -c "
import json, os

def count_words(path):
    try:
        return len(open(path).read().split())
    except: return 0

cwd = os.getcwd()
home = os.path.expanduser('~')

layers = {
    'Global AGENTS.md': count_words(f'{home}/.Codex/AGENTS.md'),
    'Project AGENTS.md': count_words(f'{cwd}/AGENTS.md'),
    'Rules': sum(count_words(os.path.join(r,f))
        for r,_,fs in os.walk(f'{cwd}/.Codex/rules')
        for f in fs if f.endswith('.md')) if os.path.isdir(f'{cwd}/.Codex/rules') else 0,
    'MEMORY.md': count_words(f'{home}/.Codex/projects/-{cwd.replace(\"/\",\"-\").replace(\"_\",\"-\").lstrip(\"-\")}/memory/MEMORY.md'),
}

# Skill descriptions (loaded at startup)
skill_words = 0
for skill_dir in [f'{cwd}/.Codex/skills', f'{home}/.Codex/skills']:
    if os.path.isdir(skill_dir):
        for d in os.listdir(skill_dir):
            sm = os.path.join(skill_dir, d, 'SKILL.md')
            if os.path.isfile(sm):
                try:
                    with open(sm) as sf:
                        for line in sf:
                            if line.startswith('description:'):
                                skill_words += len(line.split())
                                break
                except: pass
layers['Skill descriptions'] = skill_words

# MCP tool schemas (estimate)
try:
    s = json.load(open(f'{cwd}/.Codex/settings.local.json'))
    mcp_count = len(s.get('enabledMcpjsonServers', []))
    layers['MCP schemas (est)'] = mcp_count * 25 * 15  # ~25 tools x 15 words each
except: pass

total = sum(layers.values())
tokens_est = int(total * 1.3)  # words -> tokens rough ratio

print(f'{'Layer':<25} {'Words':>8} {'~Tokens':>8}')
print('─' * 43)
for name, words in sorted(layers.items(), key=lambda x: -x[1]):
    print(f'{name:<25} {words:>8,} {int(words*1.3):>8,}')
print('─' * 43)
print(f'{'TOTAL':<25} {total:>8,} {tokens_est:>8,}')
print(f'Budget used: ~{tokens_est/200000*100:.1f}% of 200K context')
"
```

### 3. Show per-call overhead

```bash
echo "=== PER-CALL HOOKS ==="
python3 -c "
import json, os

for label, path in [('Project', f'{os.getcwd()}/.Codex/settings.json'),
                     ('Global', f'{os.path.expanduser(\"~\")}/.Codex/settings.json')]:
    try:
        d = json.load(open(path))
    except: continue

    hooks = d.get('hooks', {})
    for event in ['PreToolUse', 'PostToolUse', 'UserPromptSubmit']:
        entries = hooks.get(event, [])
        for entry in entries:
            matcher = entry.get('matcher', '(all)')
            for h in entry.get('hooks', []):
                htype = h.get('type', '?')
                detail = h.get('command', h.get('prompt', ''))[:80]
                print(f'{label:8} {event:18} {matcher:20} {htype:8} {detail}')
"
```

### 4. Present compact report

Format as:

```
init-tune diagnostic
━━━━━━━━━━━━━━━━━━━

Startup: XXXms (banner: XXms, connect: XXms, observer: XXms)
Context: ~XX,XXX tokens (XX% of 200K budget)
Per-call: X command hooks (XXms) + X prompt hooks (~XXX tokens)

Top consumers:
  1. [layer] — XXXX tokens
  2. [layer] — XXXX tokens
  3. [layer] — XXXX tokens

Quick wins:
  - [actionable suggestion if any]
```

Keep it under 15 lines. No subagents needed — this is a Light-context skill.
