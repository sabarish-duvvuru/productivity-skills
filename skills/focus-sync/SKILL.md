---
name: Focus Sync
description: Use when detecting macOS Focus mode and sync to shadow-prism beam, iTerm2 profile, and shadow-connect zone. Reads ~/Library/DoNotDisturb/DB/ for live state. Trigger on "focus mode", "focus sync", "what focus", "which beam", "prism sync", "work mode", "personal mode", "switch context".
icon: icon.svg
metadata:
  category: enterprise/work
  family: enterprise
  lifecycle: active
  canonical_slug: astemo-focus-sync
  icon_style: craft-category-glyph-v1
---

# focus-sync — Focus Mode ↔ Prism Beam Bridge

Reads the active macOS Focus mode and synchronizes:
- **Prism beam** (dev/professional/personal/private)
- **iTerm2 profile** (colors, badge, tab color)
- **shadow-connect zone** (enterprise/developer/personal)
- **User variables** for status bar and badges

## Focus ↔ Prism Mapping

| macOS Focus | System ID | Prism Beam | iTerm Profile | Zone | Color |
|---|---|---|---|---|---|
| Work | `com.apple.focus.work` | professional | Astemo | enterprise | #B6001A |
| Personal | `com.apple.focus.personal-time` | personal | Light | personal | #4796E4 |
| Do Not Disturb | `com.apple.donotdisturb.mode.default` | private | Shadow Zen | private | #FFC000 |
| Sleep | `com.apple.sleep.sleep-mode` | (inactive) | — | — | — |
| Driving | `com.apple.donotdisturb.mode.driving` | (inactive) | — | — | — |
| Reduce Interrupts | `com.apple.focus.reduce-interruptions` | private | Shadow Zen | private | #FFC000 |
| *(none / custom "Dev")* | — | dev | Shadow | dev | #8478CE |
| *(no focus active)* | — | dev | Shadow | dev | #8478CE |

## Step 1: Detect current Focus mode

Run the detection script:

```bash
python3 -c "
import json, os

assertions_path = os.path.expanduser('~/Library/DoNotDisturb/DB/Assertions.json')
modes_path = os.path.expanduser('~/Library/DoNotDisturb/DB/ModeConfigurations.json')

try:
    with open(assertions_path) as f:
        assertions = json.load(f)
    with open(modes_path) as f:
        modes = json.load(f)
except FileNotFoundError:
    print('focus=off beam=dev zone=dev profile=Shadow')
    exit(0)

mode_map = {}
for key, val in modes['data'][0]['modeConfigurations'].items():
    mode_map[key] = val.get('mode', {}).get('name', key)

records = assertions.get('data', [{}])[0].get('storeAssertionRecords', [])

BEAM_MAP = {
    'Work': ('professional', 'enterprise', 'Astemo', '#B6001A'),
    'Personal': ('personal', 'personal', 'Light', '#4796E4'),
    'Do Not Disturb': ('private', 'private', 'Shadow Zen', '#FFC000'),
    'Reduce Interruptions': ('private', 'private', 'Shadow Zen', '#FFC000'),
    'Sleep': ('inactive', 'inactive', None, None),
    'Driving': ('inactive', 'inactive', None, None),
}
DEFAULT = ('dev', 'dev', 'Shadow', '#8478CE')

if not records:
    beam, zone, profile, color = DEFAULT
    focus_name = 'off'
else:
    mode_id = records[0]['assertionDetails']['assertionDetailsModeIdentifier']
    focus_name = mode_map.get(mode_id, mode_id.split('.')[-1])
    beam, zone, profile, color = BEAM_MAP.get(focus_name, DEFAULT)

print(f'focus={focus_name} beam={beam} zone={zone} profile={profile} color={color}')
"
```

## Step 2: Apply to iTerm2

If the detected beam differs from the current session context, switch:

```bash
# Set iTerm2 user variables (shell integration required)
iterm2_set_user_var shadow_beam "$BEAM"
iterm2_set_user_var shadow_zone "$ZONE"

# Switch profile via escape sequence
printf "\033]1337;SetProfile=%s\a" "$PROFILE"

# Set tab color
case "$BEAM" in
    professional) printf "\033]6;1;bg;red;brightness;182\a\033]6;1;bg;green;brightness;0\a\033]6;1;bg;blue;brightness;26\a" ;;
    personal)     printf "\033]6;1;bg;red;brightness;71\a\033]6;1;bg;green;brightness;150\a\033]6;1;bg;blue;brightness;228\a" ;;
    dev)          printf "\033]6;1;bg;red;brightness;132\a\033]6;1;bg;green;brightness;122\a\033]6;1;bg;blue;brightness;206\a" ;;
    private)      printf "\033]6;1;bg;red;brightness;255\a\033]6;1;bg;green;brightness;192\a\033]6;1;bg;blue;brightness;0\a" ;;
esac

# Set badge
printf "\e]1337;SetBadgeFormat=%s\a" "$(echo -n "$BEAM" | base64)"
```

## Step 3: Lock shadow-connect zone

When Focus is Work, lock to enterprise zone — prevent accidental personal/dev tool activation:
- Skill routing: only astemo-*, fh4s-*, enterprise skills
- Identity: sduvvuru (Astemo M365)
- Chrome profile: "Sabarish"
- **GLM-001**: GLM/Z.AI BLOCKED in work focus. All traffic treated as RED tier. Route to local Ollama or Codex only.

When Focus is Personal, lock to personal zone:
- Skill routing: shadow-*, mesh-*, personal skills
- Identity: ishuru / theleostark
- Chrome profile: personal
- **GLM-001**: GLM allowed for GREEN-tier tasks. Run `glm-gate` before routing to GLM.

## Creating Custom Focus Modes

Two custom Focus modes are recommended but not yet created:

### Dev Focus (missing — maps to prism:dev)
Create in **Settings > Focus > + > Custom**:
- Name: "Dev"
- Icon: laptopcomputer, purple
- Allow: iTerm2, VS Code, GitHub Desktop, Slack (dev channels only)
- Silence: Mail, Teams, Calendar
- Schedule: manual activation

### Private Focus (refine existing DND)
The current DND works but a dedicated Private focus would be cleaner:
- Name: "Private"
- Icon: lock.fill, amber/orange
- Allow: Messages, Phone only
- Silence: everything else
- Schedule: manual

To create via Shortcuts automation:
```
Shortcut: "Activate Dev Focus"
  → Set Focus: Dev (On)
  → Run Shell Script: printf "\033]1337;SetProfile=Shadow\a"
```

## Automation via Shortcuts

Create these Focus automations in the Shortcuts app:

| When Focus... | Do |
|---|---|
| Work turns on | Run: `echo -ne "\033]1337;SetProfile=Astemo\a" > /dev/ttys000` |
| Work turns off | Run: `echo -ne "\033]1337;SetProfile=Shadow\a" > /dev/ttys000` |
| Personal turns on | Run: `echo -ne "\033]1337;SetProfile=Light\a" > /dev/ttys000` |
| DND turns on | Run: `echo -ne "\033]1337;SetProfile=Shadow Zen\a" > /dev/ttys000` |

Note: The `/dev/ttys000` approach only reaches the first terminal. For all terminals, use the iTerm2 Python API AutoLaunch script (see focus-iterm-bridge skill).

## Pipeline

```
Input (trigger: focus mode, focus sync, what focus, prism sync, work mode)
  ↓
Detect Focus Mode (read ~/Library/DoNotDisturb/DB/Assertions.json + ModeConfigurations.json)
  ↓
Map to Prism State (beam, zone, profile, color)
  ↓
Apply:
  ├── iTerm2 profile switch (escape sequence)
  ├── Tab color (RGB escape)
  ├── Badge text (base64-encoded)
  ├── shadow-connect zone lock
  └── GLM tier policy (RED in Work, GREEN in Personal)
  ↓
Artifact Generation
  └── Inline status output (no persistent artifact — this is a runtime sync)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Detect focus → sync to all surfaces (iTerm, prism, zone) | Any focus trigger |
| `detect` | Show current focus mode without syncing | `what focus`, `which focus` |
| `sync` | Force-sync current focus to all surfaces | `focus sync`, `prism sync` |
| `lock <beam>` | Force-lock to a specific beam regardless of macOS focus | `lock work`, `lock dev` |
| `unlock` | Remove lock, return to macOS focus auto-detection | `unlock focus` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Inline status | Chat output | Ephemeral — no persistent artifact needed |

## Fallback Chain

1. **Primary:** Read `~/Library/DoNotDisturb/DB/` → map → apply to iTerm2 + shadow-connect
2. **DoNotDisturb DB not found:** Default to `beam=dev zone=dev profile=Shadow`; note that focus detection is unavailable
3. **iTerm2 escape sequences fail:** Report; suggest running from iTerm2 shell (not from other terminals)
4. **shadow-connect zone lock fails:** Apply iTerm profile only; note zone lock failure
5. **Focus mode not in mapping table:** Default to dev beam; note unrecognized focus mode
6. **Last resort:** Report focus mode as text only; note that profile/zone sync was not applied

## Prerequisites

- macOS with Focus mode support ( Ventura+)
- `~/Library/DoNotDisturb/DB/` readable (system directory, no special permissions needed)
- iTerm2 with shell integration for escape sequences
- `python3` for detection script
- shadow-connect running for zone lock (optional)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Assertions.json not found | Default to dev beam; report that macOS Focus DB is unavailable |
| ModeConfigurations parse error | Default to dev beam; report parse failure; suggest macOS restart |
| iTerm2 profile not found | Report; list available profiles; skip profile switch; apply color only |
| Tab color escape ignored | Running from non-iTerm terminal; report; suggest running from iTerm shell |
| shadow-connect not running | Skip zone lock; apply visual changes only; note zone lock pending |
| Custom focus mode not in mapping | Default to dev beam; suggest adding to BEAM_MAP in skill script |

## Contract

- **Read-only detection.** focus-sync reads macOS Focus state; it does not change the Focus mode itself.
- **Zone lock is safety-critical.** When Focus is Work, GLM/Z.AI is RED tier. This is non-negotiable.
- **No credential exposure.** Focus state output contains no secrets.
- **No persistent artifacts needed.** Focus state is ephemeral and runtime-only. No reports written to disk.
- **Do not** override Work focus zone lock to allow personal/dev tools. The zone lock exists to prevent data leakage.
- **Do not** change iTerm profile for terminals other than the current session.
- **Do not** create or modify macOS Focus modes from this skill. That requires user action in System Settings.

## Quick Check (one-liner)

```bash
python3 -c "import json,os;a=json.load(open(os.path.expanduser('~/Library/DoNotDisturb/DB/Assertions.json')));m=json.load(open(os.path.expanduser('~/Library/DoNotDisturb/DB/ModeConfigurations.json')));r=a.get('data',[{}])[0].get('storeAssertionRecords',[]);print(m['data'][0]['modeConfigurations'][r[0]['assertionDetails']['assertionDetailsModeIdentifier']]['mode']['name'] if r else 'off')"
```
