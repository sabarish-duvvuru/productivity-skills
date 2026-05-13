---
name: Headless Calendar Native
description: 'Use when the user needs: Direct macOS Calendar.app bridge for querying local calendars, checking private availability, and creating quick local calendar events.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-calendar-native
  icon_style: craft-category-glyph-v1
---

# hl-calendar-native

Direct bridge to the macOS Calendar.app for querying and managing local events.

## When to Use
- When querying events from the "Private" space (iCloud/Local).
- When you need to check availability without network/API overhead.
- When adding quick reminders that should appear in the Calendar UI.

## Core Capabilities

### 1. List Events for Today
```bash
osascript -e 'set today to (current date)
set time of today to 0
set tomorrow to today + (24 * 60 * 60)
tell application "Calendar"
    set eventList to ""
    repeat with theCalendar in calendars
        set theEvents to (every event of theCalendar whose start date is greater than or equal to today and start date is less than tomorrow)
        repeat with theEvent in theEvents
            set eventList to eventList & (summary of theEvent) & " [" & (name of theCalendar) & "]\n"
        end repeat
    end repeat
    return eventList
end tell'
```

### 2. Create Quick Event
```bash
osascript -e 'tell application "Calendar"
    tell calendar "Calendar"
        make new event with properties {summary:"Shadow Handoff", start date:(current date) + 3600, end date:(current date) + 7200}
    end tell
end tell'
```

## Key Decisions
- **Calendar Selection**: Default to "Calendar" or "Home". If multiple calendars exist, list them first.
- **Privacy**: Native calendar access often contains sensitive family/personal info. Do not export the full calendar unless necessary.

## Crystallized From
- Session: 2026-04-19
- Original task: "continue to improve" -> Expanding the hl-apple ecosystem.
- Pattern extracted: Native Calendar.app event retrieval via osascript.


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

