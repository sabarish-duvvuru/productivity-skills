---
name: 'Headless ChatGPT'
description: 'Use when the user needs: ChatGPT API access for when Codex limits hit. Routes through ChatGPT Pro subscription for continued work when weekly Codex quota exhausted.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-chatgpt
  icon_style: craft-category-glyph-v1
---

# hl-chatgpt — ChatGPT Pro API Access

**Purpose:** Bypass lane when Codex weekly limits hit. Use ChatGPT Pro $200 subscription directly via API for continued development work.

## When to Use

Trigger this skill when:
- Codex shows "weekly limit reached" or "quota exhausted"
- Need to continue coding work but Codex blocked
- Want to extract value from prepaid ChatGPT Pro subscription

## Method 1: OpenAI API Direct (preferred)

Use OpenAI Python SDK with ChatGPT Pro API key:

```bash
# Set API key (from shadow-remind or keychain)
export OPENAI_API_KEY="$(security find-generic-password -s 'openai-api-key' -w)"

# Direct API call for coding task
python3 - <<'PYTHON'
import openai
client = openai.OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",  # or gpt-4-turbo for cost savings
    messages=[
        {"role": "system", "content": "You are an expert coding assistant."},
        {"role": "user", "content": "YOUR_CODING_TASK_HERE"}
    ],
    temperature=0.3,
    max_tokens=4000
)

print(response.choices[0].message.content)
PYTHON
```

## Method 2: Browser-based (fallback when API rate limited)

Use Chrome DevTools MCP to access ChatGPT Pro via web interface:

```bash
# Navigate to ChatGPT
# Use ishuru profile (Profile 2) for dev context
mcp__chrome-devtools__navigate_page -u "https://chat.openai.com" -p "ishuru"

# Take snapshot of interface
mcp__chrome-devtools__take_snapshot

# Extract conversation content via DOM
mcp__chrome_devtools__evaluate_script -f '(element) => { return element.innerText; }'
```

## Model Selection

**ChatGPT Pro models available:**
- `gpt-4o` — Best for complex reasoning, long context (128K)
- `gpt-4-turbo` — Faster, cheaper, good for most coding
- `gpt-4o-mini` — Fastest, cheapest, simple tasks

**Cost optimization:**
```python
# Use gpt-4-turbo for 60% cost savings over gpt-4o
model="gpt-4-turbo"  # $0.01/1K tokens vs $0.025/1K tokens
```

## Task Type Routing

Match Codex task patterns to ChatGPT models:

| Codex Task | ChatGPT Model | Temp | Max Tokens |
|------------|----------------|------|------------|
| editor_native_coding | gpt-4-turbo | 0.2 | 4000 |
| implementation_coding | gpt-4o | 0.3 | 8000 |
| deep_review | gpt-4o | 0.3 | 16000 |
| brainstorm | gpt-4o | 0.85 | 4000 |
| web_search | gpt-4o-mini | 0.5 | 2000 |

## Session Tracking

Track ChatGPT usage via session-tracker hook:

```bash
# Log ChatGPT API call
python3 /Volumes/☯Duality/ShadowArchive/03-agent-roots/subscription-maximize/cost-tracker.py \
  track chatgpt-pro-200 150 0.15 implementation_coding
```

**Parameters:**
- subscription_id: `chatgpt-pro-200`
- tokens: actual token count from API response
- cost_usd: calculate based on model pricing
- task_type: match Codex task type

## API Key Management

Store API key securely in macOS keychain:

```bash
# Add API key to keychain (one-time setup)
security add-generic-password -s "openai-api-key" -a "sdluffy" -w "sk-..."

# Retrieve API key
security find-generic-password -s "openai-api-key" -w
```

## Cost Tracking

ChatGPT Pro $200 subscription includes:
- Weekly quota resets (check current status in ChatGPT settings)
- Higher limits than free/plus tiers
- Priority access during high demand

**Track real usage:**
```bash
# Check subscription status
python3 subscription-analyzer.py analyze chatgpt-pro-200

# View session history
python3 session-tracker.py analyze 7

# Cost report
python3 swarm-orchestrator.py report
```

## Integration with Existing Routing

ChatGPT Pro is already configured in task routing matrix:

```bash
# Get optimal routing for current task
python3 task-router.py suggest

# Force ChatGPT Pro routing
python3 task-router.py route implementation_coding --primary chatgpt-pro-200
```

## Browser Profile Routing

Use correct Chrome profile:
- **ishuru** (Profile 2): Dev context, GitHub SSO, open source
- **sdluffy** (Default): Personal, iCloud, Proton
- **sduvvuru** (Profile 5): Work, Astemo, M365

For coding tasks, prefer `ishuru` profile (already logged into GitHub/dev accounts).

## Output Format

Save ChatGPT responses to structured format:

```json
{
  "id": "chatgpt-{timestamp}",
  "model": "gpt-4o",
  "task_type": "implementation_coding",
  "tokens_used": 1234,
  "cost_usd": 0.031,
  "response": "...",
  "timestamp": "2026-04-19T23:55:00Z",
  "subscription": "chatgpt-pro-200"
}
```

## When Codex Returns

Once weekly quota resets:
- Switch back to Codex for editor-native coding
- Use ChatGPT Pro for overflow/burst work
- Continue tracking usage across both providers

**Monitor quota status:**
```bash
# Check if quota reset happened
python3 subscription-analyzer.py analyze chatgpt-pro-200
```

## Quick Reference

```bash
# Direct API call (Python)
python3 -c "import openai; client = openai.OpenAI(); print(client.chat.completions.create(model='gpt-4-turbo', messages=[{'role':'user','content':'YOUR_TASK'}]).choices[0].message.content)"

# Check subscription health
python3 subscription-analyzer.py analyze chatgpt-pro-200

# Log usage
python3 cost-tracker.py track chatgpt-pro-200 1000 0.10 implementation_coding

# Get routing recommendation
python3 task-router.py suggest
```

## Cost Optimization Tips

1. **Use gpt-4-turbo** for 60% cost savings (good enough for most coding)
2. **Limit context** to only necessary files (reduces input tokens)
3. **Lower max_tokens** for simple tasks (2000-4000 usually enough)
4. **Cache responses** for repeated questions (use shadow-vision-extract pattern)
5. **Weekly quota monitoring** — check status before heavy usage days

**Strategy:** Extract maximum value from prepaid ChatGPT Pro subscription while waiting for Codex weekly reset.


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

