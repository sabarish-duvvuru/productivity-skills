---
name: Headless Cu Skills
description: 'Use when the user needs: Headless computer-use primitives for GUI automation, screenshots, vision analysis, and macOS app interaction when no CLI or API exists. Uses cua-driver for backgrounded per-PID automation — no focus theft, no cursor warp.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-cu-skills
  icon_style: craft-category-glyph-v1
---

# hl-cu-skills — Background GUI Automation via cua-driver

Drive macOS apps in the background using `cua-driver` (v0.1.2). No focus theft, no cursor warp, no Space switching.

## When to Use
- When a task requires interacting with apps that lack a CLI or API
- When performing visual debugging or verification of UI states
- When the operator asks for "computer use" or "GUI automation"
- **Preferred over** `cliclick`, `osascript` GUI commands, and `browser-surface-bridge` for all GUI automation

## Prerequisites
- `cua-driver` installed: `/Users/sdluffy/.local/bin/cua-driver`
- `CuaDriver.app` in `/Applications`
- Accessibility + Screen Recording permissions granted
- Daemon running: `open -n -g -a CuaDriver --args serve`

## The No-Foreground Contract

**The user's frontmost app MUST NOT change.** This is the entire reason cua-driver exists.

Forbidden (always steal focus):
- `open -a <App>` — ALL forms of `open` activate via LaunchServices
- `osascript 'tell app X to activate'`
- `cliclick` — moves the real cursor
- `osascript` mutating frontmost state

Always use instead:
- `cua-driver launch_app({bundle_id})` — hidden launch, no activation
- `cua-driver click({pid, element_index})` — per-PID AX dispatch
- `cua-driver press_key({pid, key})` — per-PID keyboard events

## Core Loop: Observe → Reason → Act → Verify

```
1. launch_app({bundle_id}) → get pid + window_id
2. get_window_state({pid, window_id}) → AX tree + screenshot
3. [act] → click/type/scroll/set_value via element_index
4. get_window_state({pid, window_id}) → verify action landed
```

**Snapshot before AND after every action.** Element indices are cached per `(pid, window_id)` and invalidated on each snapshot.

## CLI Usage

```bash
# Start daemon (required for element_index workflows)
open -n -g -a CuaDriver --args serve

# Launch app hidden
cua-driver launch_app '{"bundle_id": "com.apple.finder"}'

# Snapshot window
cua-driver get_window_state '{"pid": 123, "window_id": 456}' --screenshot-out-file /tmp/shot.jpg

# Click by element_index (AX — works backgrounded)
cua-driver click '{"pid": 123, "window_id": 456, "element_index": 14}'

# Click by pixel (for canvas/WebView surfaces)
cua-driver click '{"pid": 123, "x": 500, "y": 300}'

# Type text
cua-driver type_text '{"pid": 123, "text": "hello world"}'

# Press key
cua-driver press_key '{"pid": 123, "key": "return"}'

# Hotkey
cua-driver hotkey '{"pid": 123, "keys": ["cmd", "c"]}'

# Read page text (browsers/Electron)
cua-driver page '{"pid": 123, "window_id": 456, "action": "get_text"}'

# Query DOM
cua-driver page '{"pid": 123, "window_id": 456, "action": "query_dom", "css_selector": "a[href]", "attributes": ["href"]}'

# Set field value directly (works on minimized windows)
cua-driver set_value '{"pid": 123, "window_id": 456, "element_index": 7, "value": "new value"}'

# Scroll
cua-driver scroll '{"pid": 123, "direction": "down", "amount": 3}'

# Stop daemon
cua-driver stop
```

## Web App Interaction (Chrome, Safari, Electron)

For Chrome/Electron apps, AX trees can be sparse. Use `page` tool for DOM access:

```bash
# Read all text on page
cua-driver page '{"pid": 70959, "window_id": 37350, "action": "get_text"}'

# Extract all links
cua-driver page '{"pid": 70959, "window_id": 37350, "action": "query_dom", "css_selector": "a[href]", "attributes": ["href", "textContent"]}'

# Execute arbitrary JS
cua-driver page '{"pid": 70959, "window_id": 37350, "action": "execute_javascript", "javascript": "document.title"}'
```

## Capture Modes

| Mode | Returns | Use for |
|---|---|---|
| `som` (default) | AX tree + screenshot | General purpose — element_index + visual |
| `ax` | AX tree only | When screenshot isn't needed |
| `vision` | Screenshot only | When AX tree isn't useful (games, canvas) |

## Migration from Old Primitives

| Old (focus-unsafe) | New (cua-driver) |
|---|---|
| `screencapture -x /tmp/shot.png` | `cua-driver get_window_state --screenshot-out-file` |
| `cliclick c:500,500` | `cua-driver click '{pid, element_index}'` or `'{pid, x, y}'` |
| `osascript keystroke "hello"` | `cua-driver type_text '{pid, text}'` |
| `osascript key code 36` | `cua-driver press_key '{pid, key: "return"}'` |
| `osascript activate app X` | `cua-driver launch_app '{bundle_id}'` (hidden) |
| `browser-surface-bridge` JS inject | `cua-driver page '{action: execute_javascript}'` |

## Error Recovery

| Error | Fix |
|---|---|
| `No cached AX state` | Re-run `get_window_state` with same `(pid, window_id)` |
| `Invalid element_index` | Re-snapshot, use fresh indices |
| `window_id belongs to pid P, not ...` | Use `list_windows({pid})` for correct windows |
| Screenshot too small | Check `capture_mode` — default `som` should include screenshot |
| `Accessibility permission not granted` | System Settings → Privacy → Accessibility → CuaDriver.app |

## Crystallized From
- Session: 2026-04-19 (cliclick + osascript primitives)
- Session: 2026-05-03 (migrated to cua-driver v0.1.2 — background GUI automation with zero focus theft)
