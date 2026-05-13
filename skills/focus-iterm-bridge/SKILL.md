---
name: Focus iTerm Bridge
description: 'Use when the user needs: iTerm2 automation bridge — dynamic profile switching, color theming, badge/status bar, user variables, AutoLaunch scripts, Focus-reactive terminal. Trigger on "iterm colors", "iterm profile switch", "terminal theme", "badge text", "status bar component", "iterm autolaunch", "terminal personalize", "iterm escape", "tab color".'
icon: icon.svg
metadata:
  category: enterprise/work
  family: enterprise
  lifecycle: active
  canonical_slug: focus-iterm-bridge
  icon_style: craft-category-glyph-v1
---

# focus-iterm-bridge — iTerm2 Automation Bridge

Programmatic control of iTerm2 from Codex — profiles, colors, badges, status bar, and Focus mode reactivity.

## Escape Sequence Quick Reference

These work from any shell — no Python API needed:

```bash
# Switch profile
printf "\033]1337;SetProfile=%s\a" "Astemo"

# Set tab color (RGB 0-255)
printf "\033]6;1;bg;red;brightness;182\a"
printf "\033]6;1;bg;green;brightness;0\a"
printf "\033]6;1;bg;blue;brightness;26\a"

# Reset tab color
printf "\033]6;1;bg;*;default\a"

# Set badge
printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n 'ASTEMO' | base64)"

# Set user variable (readable in badge as \(user.xxx))
printf "\033]1337;SetUserVar=%s=%s\007" "program" "$(echo -n FH4S | base64)"

# Set tab/window title
printf "\033]0;ASTEMO — FH4S\007"

# Set background image
printf "\033]1337;SetBackgroundImageFile=%s\a" "/path/to/image.png"
```

## Shell Integration Utilities

With `~/.iterm2_shell_integration.zsh` sourced:

```bash
# it2setcolor — change colors live
it2setcolor tab B6001A              # Astemo Red
it2setcolor foreground ffffff background 1e1e24
it2setcolor --reset                 # back to profile defaults

# it2profile — switch profile
it2profile -s "Astemo"              # switch
it2profile -g                       # get current name

# iterm2_set_user_var — set badge/status bar variables
iterm2_set_user_var program "FH4S"
iterm2_set_user_var beam "professional"
iterm2_set_user_var zone "enterprise"
```

## Prism Beam Color Palette

| Beam | Hex | RGB | Use |
|------|-----|-----|-----|
| professional | #B6001A | 182,0,26 | Tab color, cursor, badge |
| dev | #8478CE | 132,122,206 | Tab color, cursor |
| personal | #4796E4 | 71,150,228 | Tab color |
| private | #FFC000 | 255,192,0 | Tab color |
| agent | #00B050 | 0,176,80 | Badge accent |

## Apply Beam to Current Session

```bash
apply_beam() {
    local beam="$1"
    case "$beam" in
        professional)
            printf "\033]6;1;bg;red;brightness;182\a\033]6;1;bg;green;brightness;0\a\033]6;1;bg;blue;brightness;26\a"
            printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n 'ASTEMO' | base64)"
            ;;
        dev)
            printf "\033]6;1;bg;red;brightness;132\a\033]6;1;bg;green;brightness;122\a\033]6;1;bg;blue;brightness;206\a"
            printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n 'SHADOW' | base64)"
            ;;
        personal)
            printf "\033]6;1;bg;red;brightness;71\a\033]6;1;bg;green;brightness;150\a\033]6;1;bg;blue;brightness;228\a"
            printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n 'LIGHT' | base64)"
            ;;
        private)
            printf "\033]6;1;bg;red;brightness;255\a\033]6;1;bg;green;brightness;192\a\033]6;1;bg;blue;brightness;0\a"
            printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n '' | base64)"
            ;;
    esac
    printf "\033]1337;SetUserVar=%s=%s\007" "shadow_beam" "$(echo -n "$beam" | base64)"
}
```

## Dynamic Profiles

Location: `~/Library/Application Support/iTerm2/DynamicProfiles/`

JSON files here are auto-loaded (hot-reload, no restart). Use `Dynamic Profile Parent Name` to inherit and override selectively.

### Prism Profile Template

```json
{
  "Profiles": [
    {
      "Name": "Shadow",
      "Guid": "shadow-dev-profile",
      "Dynamic Profile Parent Name": "Default",
      "Badge Text": "\\(user.shadow_beam)",
      "Tab Color": { "Red Component": 0.518, "Green Component": 0.478, "Blue Component": 0.808 },
      "Use Tab Color": true,
      "Tags": ["shadow", "dev"],
      "Working Directory": "/Volumes/SHADOW",
      "Custom Directory": "Recycle"
    },
    {
      "Name": "Astemo",
      "Guid": "astemo-enterprise-profile",
      "Dynamic Profile Parent Name": "Default",
      "Badge Text": "ASTEMO \\(user.program)",
      "Badge Color": { "Red Component": 0.714, "Green Component": 0.0, "Blue Component": 0.102, "Alpha Component": 0.5 },
      "Tab Color": { "Red Component": 0.714, "Green Component": 0.0, "Blue Component": 0.102 },
      "Use Tab Color": true,
      "Tags": ["astemo", "enterprise"],
      "Working Directory": "/Volumes/SHADOW/ASTEMO",
      "Custom Directory": "Recycle"
    },
    {
      "Name": "Light",
      "Guid": "light-personal-profile",
      "Dynamic Profile Parent Name": "Default",
      "Badge Text": "\\(user.shadow_beam)",
      "Tab Color": { "Red Component": 0.278, "Green Component": 0.588, "Blue Component": 0.894 },
      "Use Tab Color": true,
      "Tags": ["light", "personal"]
    },
    {
      "Name": "Shadow Zen",
      "Guid": "shadow-zen-private-profile",
      "Dynamic Profile Parent Name": "Default",
      "Badge Text": "",
      "Tab Color": { "Red Component": 1.0, "Green Component": 0.753, "Blue Component": 0.0 },
      "Use Tab Color": true,
      "Tags": ["zen", "private"],
      "Background Color": { "Red Component": 0.08, "Green Component": 0.08, "Blue Component": 0.10 }
    }
  ]
}
```

## Automatic Profile Switching (built-in)

In **Profiles > Advanced > Automatic Profile Switching**, add path rules:

```
*@*:/Volumes/SHADOW/ASTEMO*  →  Astemo
*@*:/Volumes/SHADOW/shadow-lab*  →  Shadow
*@*:~/*  →  Light
```

Requires shell integration. Profile switches when you `cd` to matching paths.

## iTerm2 AutoLaunch Script (Focus-reactive)

Place in `~/Library/Application Support/iTerm2/Scripts/AutoLaunch/`:

```python
#!/usr/bin/env python3
"""focus-bridge.py — sync macOS Focus to iTerm2 profiles."""
import iterm2
import json
import os
import asyncio

BEAM_MAP = {
    "Work": ("professional", "Astemo", iterm2.Color(182, 0, 26)),
    "Personal": ("personal", "Light", iterm2.Color(71, 150, 228)),
    "Do Not Disturb": ("private", "Shadow Zen", iterm2.Color(255, 192, 0)),
    "Reduce Interruptions": ("private", "Shadow Zen", iterm2.Color(255, 192, 0)),
}
DEFAULT = ("dev", "Shadow", iterm2.Color(132, 122, 206))

ASSERTIONS = os.path.expanduser("~/Library/DoNotDisturb/DB/Assertions.json")
MODES = os.path.expanduser("~/Library/DoNotDisturb/DB/ModeConfigurations.json")

def get_focus():
    try:
        with open(ASSERTIONS) as f:
            a = json.load(f)
        with open(MODES) as f:
            m = json.load(f)
        mode_map = {k: v.get("mode", {}).get("name", k)
                    for k, v in m["data"][0]["modeConfigurations"].items()}
        records = a.get("data", [{}])[0].get("storeAssertionRecords", [])
        if records:
            mid = records[0]["assertionDetails"]["assertionDetailsModeIdentifier"]
            return mode_map.get(mid, "off")
    except Exception:
        pass
    return "off"

async def main(connection):
    last_focus = None
    while True:
        focus = get_focus()
        if focus != last_focus:
            last_focus = focus
            beam, profile_name, tab_color = BEAM_MAP.get(focus, DEFAULT)
            app = await iterm2.async_get_app(connection)
            for window in app.terminal_windows:
                for tab in window.tabs:
                    for session in tab.sessions:
                        change = iterm2.LocalWriteOnlyProfile()
                        change.set_tab_color(tab_color)
                        change.set_use_tab_color(True)
                        await session.async_set_profile_properties(change)
        await asyncio.sleep(10)  # poll every 10s

iterm2.run_forever(main)
```

## Custom Status Bar Component

Register via AutoLaunch to show current beam in the status bar:

```python
#!/usr/bin/env python3
"""beam-status.py — show prism beam in iTerm2 status bar."""
import iterm2

async def main(connection):
    component = iterm2.StatusBarComponent(
        short_description="Prism Beam",
        detailed_description="Current shadow-prism beam from Focus mode",
        knobs=[],
        exemplar="dev",
        update_cadence=None,
        identifier="com.shadow.prism-beam"
    )

    @iterm2.StatusBarRPC
    async def coro(knobs, beam=iterm2.Reference("user.shadow_beam")):
        ICONS = {"professional": "🔴", "dev": "🟣", "personal": "🔵", "private": "🟡"}
        if beam:
            return f"{ICONS.get(beam, '⚪')} {beam}"
        return "⚪ unknown"

    await component.async_register(connection, coro, timeout=None)

iterm2.run_forever(main)
```

## .zshrc Integration

Add to `~/.zshrc` for variable persistence across restarts:

```bash
# Shadow Prism — set beam vars on shell init
if [[ "$TERM_PROGRAM" == "iTerm.app" ]]; then
    _focus=$(python3 -c "
import json,os
a=json.load(open(os.path.expanduser('~/Library/DoNotDisturb/DB/Assertions.json')))
m=json.load(open(os.path.expanduser('~/Library/DoNotDisturb/DB/ModeConfigurations.json')))
r=a.get('data',[{}])[0].get('storeAssertionRecords',[])
print(m['data'][0]['modeConfigurations'][r[0]['assertionDetails']['assertionDetailsModeIdentifier']]['mode']['name'] if r else 'off')
" 2>/dev/null || echo "off")

    case "$_focus" in
        Work)    _beam=professional ;;
        Personal) _beam=personal ;;
        "Do Not Disturb"|"Reduce Interruptions") _beam=private ;;
        *)       _beam=dev ;;
    esac

    iterm2_set_user_var shadow_beam "$_beam"

    # Auto-set program if in ASTEMO
    [[ "$PWD" == /Volumes/SHADOW/ASTEMO* ]] && iterm2_set_user_var program "FH4S"
fi
```

## Shortcuts Automation (create manually)

| Trigger | Action |
|---|---|
| Work Focus ON | Run Shell Script: `it2setcolor tab B6001A && iterm2_set_user_var shadow_beam professional` |
| Work Focus OFF | Run Shell Script: `it2setcolor tab 8478CE && iterm2_set_user_var shadow_beam dev` |
| Personal Focus ON | Run Shell Script: `it2setcolor tab 4796E4 && iterm2_set_user_var shadow_beam personal` |
| DND Focus ON | Run Shell Script: `it2setcolor tab FFC000 && iterm2_set_user_var shadow_beam private` |

Note: Shortcuts shell scripts run in a non-interactive shell without iTerm context. The AutoLaunch polling script is more reliable for reactive switching.
