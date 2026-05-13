---
name: Headless Safari Tabs
description: 'Use when the user needs: Headless Safari tab extraction and management using AppleScript for browser context retrieval, tab search, and research handoffs.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-safari-tabs
  icon_style: craft-category-glyph-v1
---

# hl-safari-tabs

Headless extraction and management of Safari windows and tabs using AppleScript.

## When to Use
- When you need to retrieve context from the operator's open browser tabs.
- When you need to find a specific URL or page title across all Safari windows.
- When performing research handoffs between the agent and the browser.

## Core Capabilities

### 1. List All Tabs
Extracts URL and Title for every tab in every window.
```bash
osascript -e 'set tabList to ""
tell application "Safari"
    set windowCount to count windows
    repeat with w from 1 to windowCount
        set currentWindow to window w
        set tabCount to count tabs of currentWindow
        repeat with t from 1 to tabCount
            set currentTab to tab t of currentWindow
            set tabList to tabList & (URL of currentTab) & " | " & (name of currentTab) & "\n"
        end repeat
    end repeat
end tell
return tabList'
```

### 2. Search Tabs
Find tabs matching a keyword in the URL or title.
```bash
# Example: Find all GitHub tabs
osascript -e 'tell application "Safari" to get every tab of every window whose URL contains "github.com"'
```

### 3. Focus or Close Tab
```bash
# Focus a tab by URL
osascript -e 'tell application "Safari"
    set targetURL to "https://github.com"
    repeat with w in windows
        repeat with t in tabs of w
            if URL of t contains targetURL then
                set current tab of w to t
                set index of w to 1
                activate
                return
            end if
        end repeat
    end repeat
end tell'
```

## Key Decisions
- **Non-Invasive**: Read-only extraction is the default. Only focus or close when explicitly requested.
- **Privacy**: Be mindful of sensitive URLs (banking, private docs) when logging tab lists.

## Crystallized From
- Session: 2026-04-19
- Original task: "continue to improve" -> Expanding the hl-apple ecosystem.
- Pattern extracted: High-fidelity Safari state retrieval via osascript.


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

