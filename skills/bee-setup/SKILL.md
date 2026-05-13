---
name: Bee Setup
description: 'Use when the user needs: Full Bee AI setup -- CLI install, login, proxy configuration on FRIDAY gateway, project setup, end-to-end verification.'
icon: icon.svg
user_invocable: true
metadata:
  category: workflow/process
  family: bee
  lifecycle: active
  canonical_slug: bee-setup
  icon_style: craft-category-glyph-v1
---

# Bee Setup

Complete end-to-end setup for Bee AI integration across the mesh.

## When to use

- First-time Bee setup on this machine or FRIDAY
- After reinstalling the CLI or resetting credentials
- When proxy needs to be reconfigured on FRIDAY
- After Bee app updates that change API behavior

## Procedure

### 1. Prerequisites Check

```bash
node --version
npm config get prefix
ssh -o ConnectTimeout=5 -o BatchMode=yes friday-mac-local echo ok
```

### 2. Install / Update Bee CLI

```bash
npm install -g @beeai/cli
bee version
```

### 3. Enable Developer Mode on iPhone

Instruct the user:
1. Open the Bee app on your iPhone
2. Go to Settings
3. Tap the app version number 5 times
4. Developer mode should now be enabled (look for Developer section)

### 4. Login

```bash
# Clear any stale sessions
security delete-generic-password -s "bee-cli" 2>/dev/null
bee login
```

This displays an authentication URL. The user must open it on their phone and approve.

Verify:
```bash
bee status
```

Should show "Logged in" with the production API URL.

### 5. Test Local Proxy

```bash
bee proxy --port 8787 &
sleep 2
curl -s http://127.0.0.1:8787/v1/me | python3 -m json.tool
kill %1
```

### 6. Configure Proxy on FRIDAY (Always-On Gateway)

a. Install bee CLI on FRIDAY:
```bash
ssh friday-mac-local 'npm install -g @beeai/cli && bee version'
```

b. Login on FRIDAY (user must approve on phone):
```bash
ssh -t friday-mac-local 'bee login'
```

c. Verify:
```bash
ssh friday-mac-local 'bee status'
```

d. Create LaunchAgent on FRIDAY:
```bash
ssh friday-mac-local 'cat > ~/Library/LaunchAgents/com.sdluffy.bg.bee.proxy.login.plist << "PLIST"
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.sdluffy.bg.bee.proxy.login</string>
  <key>ProgramArguments</key>
  <array>
    <string>/Users/leo/.npm-global/bin/bee</string>
    <string>proxy</string>
    <string>--port</string>
    <string>8787</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
  <key>StandardOutPath</key>
  <string>/tmp/bee-proxy.log</string>
  <key>StandardErrorPath</key>
  <string>/tmp/bee-proxy.err</string>
</dict>
</plist>
PLIST'
```

e. Load the LaunchAgent:
```bash
ssh friday-mac-local 'launchctl load ~/Library/LaunchAgents/com.sdluffy.bg.bee.proxy.login.plist'
```

f. Verify proxy on FRIDAY:
```bash
ssh friday-mac-local 'curl -s http://127.0.0.1:8787/v1/me | head -c 200'
```

### 7. Configure JARVIS to Use FRIDAY Proxy

Test from JARVIS over LAN:
```bash
curl -s http://friday-mac-local:8787/v1/me | python3 -m json.tool
```

### 8. Setup bee-workflow-os (Optional)

```bash
cd ~/Projects/bee-workflow-os
cp .env.example .env
# Set BEE_PROXY_BASE_URL=http://friday-mac-local:8787 for FRIDAY proxy
pnpm install
pnpm approve-builds
pnpm dev
```

### 9. iOS Privacy Lockdown

Instruct the user to go to iPhone Settings > Bee and:
- DENY: Location, Contacts, Calendar, Health, Photos, Reminders, SMS
- ALLOW: Microphone only (required for band)
- ALLOW: Bluetooth (required for band connection)

### 10. End-to-End Verification

```bash
bee me --json
bee conversations list --limit 3 --json
bee search --query "test" --limit 1 --json
timeout 5 bee stream --json 2>/dev/null; echo "Stream test complete"
bee ping --count 3
```

Report results as a checklist:
- [ ] CLI installed and version confirmed
- [ ] Logged in successfully
- [ ] Local proxy responds
- [ ] FRIDAY proxy responds (if configured)
- [ ] Profile fetch works
- [ ] Conversations accessible
- [ ] Search functional
- [ ] Stream connects
- [ ] Ping succeeds
- [ ] iOS permissions locked down

$ARGUMENTS


## Pipeline

```
Intent → Authenticate Bee API → Extract/sync/audit → Structure output → Route artifacts
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Compact summary | Normal use |
| `full` | Complete data dump | Detailed analysis |
| `json` | Raw JSON | Programmatic use |

## Artifact Routing

- Bee data exports: `ShadowArchive/80-reports/bee-exports/`
- Audit reports: `ShadowArchive/80-reports/bee-audit-YYYY-MM-DD.md`

## Prerequisites

- Bee AI account with active session
- `bee` CLI or API access

## Contract

- Never expose Bee API tokens in output
- Respect Bee rate limits
- Export user data only with operator confirmation

