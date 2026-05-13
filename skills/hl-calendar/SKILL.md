---
name: Headless Calendar
description: 'Use when the user needs: Headless Outlook Calendar automation — view schedule, check meetings, extract attendees/agendas, navigate days. Primary: cookie export → Outlook calendar API (invisible). Fallback: Playwright browser. Trigger on "calendar", "schedule", "meetings today", "what''s on my calendar", "upcoming meetings", "free slots".'
icon: icon.svg
metadata:
  filePattern: []
  bashPattern:
  - calendar
  - schedule
  - meetings today
  - outlook calendar
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-calendar
  icon_style: craft-category-glyph-v1
  invisible_first: true
  tier: 2
---

# hl-calendar — Headless Outlook Calendar

General-purpose Outlook Calendar access. **Primary path is invisible (cookie export → API).** Browser is fallback.

## Invisible-First Paths

| Priority | Method | Focus Theft | Speed |
|---|---|---|---|
| **1** | `export-cookies.py` → Outlook calendar API | ❌ None | ~2s |
| **2** | Native macOS Calendar via AppleScript | ❌ None | ~1s |
| **3** | Playwright MCP / browser_tool | ⚠️ May steal focus | ~10s |

## Layer 1: Cookie Export → Outlook API (Primary)

```bash
# Export Outlook cookies
python3 ~/.agents/skills/browser-surface-bridge/scripts/export-cookies.py \
  -p "Profile 5" -d outlook -o /tmp/outlook-cookies.txt
```

### Operations via API

```bash
COOKIES="-b /tmp/outlook-cookies.txt"

# Today's calendar events
curl -s $COOKIES \
  "https://outlook.office.com/api/v2.0/me/calendarview?startDateTime=2026-05-04T00:00:00Z&endDateTime=2026-05-04T23:59:59Z&\$select=Subject,Start,End,Location,Organizer,IsAllDay" \
  -H "Accept: application/json"

# Upcoming events (next 7 days)
curl -s $COOKIES \
  "https://outlook.office.com/api/v2.0/me/calendarview?startDateTime=2026-05-04T00:00:00Z&endDateTime=2026-05-11T00:00:00Z&\$select=Subject,Start,End,Location,Organizer,Attendees&\$top=20" \
  -H "Accept: application/json"

# Get specific event details
curl -s $COOKIES \
  "https://outlook.office.com/api/v2.0/me/events/{id}?\$select=Subject,Body,Attendees,Location,Start,End" \
  -H "Accept: application/json"

# Clean up
rm -f /tmp/outlook-cookies.txt
```

## Layer 2: Native macOS Calendar (Instant)

```bash
# Today's events (local Calendar.app — no auth needed)
osascript -e '
tell application "Calendar"
  set today to current date
  set hours of today to 0
  set minutes of today to 0
  set seconds of today to 0
  set tomorrow to today + 1 * days
  set output to ""
  repeat with cal in calendars
    set evts to (every event of cal whose start date >= today and start date < tomorrow)
    repeat with evt in evts
      set output to output & (start date of evt as text) & " | " & (summary of evt) & " | " & (location of evt) & linefeed
    end repeat
  end repeat
  return output
end tell
'
```

**Note:** Native Calendar only shows synced calendars (iCloud, personal). Astemo M365 calendar requires Layer 1 (cookie export) or browser.

## Layer 3: Browser (Fallback)

```
browser_navigate: https://outlook.office.com/calendar/
browser_snapshot
```

### Browser Operations

| Operation | Method |
|---|---|
| View today | Default view |
| Navigate days | Click arrows / mini-calendar |
| Meeting details | Click meeting block |
| Attendees | Open meeting → snapshot |
| Scheduling assistant | New event → Scheduling Assistant tab |

## View Modes

| View | Best for |
|------|----------|
| Day | Detailed single-day schedule |
| Work week | Mon-Fri overview |
| Week | Full 7-day view |

## Error Handling

| Error | Fix |
|-------|-----|
| 401 Unauthorized | Re-export cookies |
| Empty calendar | Check date range, check profile (Profile 5 for Astemo) |
| MFA prompt | Ask operator to approve |
| Browser loading | Wait 3-5s |

## Contract

- **Never enter credentials** — assume authenticated session exists
- **Read-only calendar access** — no creating/modifying meetings without operator approval
- **Cookie files are ephemeral** — delete after use
- **Calendar data is private** — do not expose meeting details in external artifacts
