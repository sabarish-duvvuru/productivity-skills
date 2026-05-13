---
name: Bee Status
description: Use when checking bee AI connectivity, authentication, proxy, and device status.
icon: icon.svg
user_invocable: true
metadata:
  category: system/security
  family: bee
  lifecycle: active
  canonical_slug: bee-status
  icon_style: craft-category-glyph-v1
---

# Bee Status Check

Comprehensive health check of the Bee ecosystem — device, CLI, proxy, and cloud.

## When to use

- Quick check if Bee is working
- Debugging connection issues
- After device pairing changes
- Verifying proxy and API access

## Procedure

### 1. CLI Status

```bash
bee version 2>/dev/null || echo "CLI not installed"
bee status 2>/dev/null || echo "Not logged in"
```

### 2. Device Connectivity

Check if the Bee device is visible via BLE from this Mac:

```bash
cd ~/Spaces/Dev/bee 2>/dev/null && source .venv/bin/activate 2>/dev/null && python3 -c "
import asyncio
from bleak import BleakScanner
async def scan():
    devices = await BleakScanner.discover(timeout=10, return_adv=True)
    for d, adv in devices.values():
        name = d.name or adv.local_name or ''
        mfr = adv.manufacturer_data or {}
        if 'bee' in name.lower() or 0xBEEE in mfr:
            print(f'Bee found: {name} RSSI={adv.rssi}dBm addr={d.address}')
            if adv.rssi < -80: print('  WARNING: Weak signal')
            elif adv.rssi < -60: print('  Signal: Fair')
            else: print('  Signal: Good')
            return
    print('Bee device NOT found in BLE scan')
asyncio.run(scan())
" 2>/dev/null || echo "BLE scan not available (need ~/Spaces/Dev/bee/.venv with bleak)"
```

### 3. API Access

```bash
bee me --json 2>/dev/null | python3 -c "
import json,sys
try:
    d=json.load(sys.stdin)
    print(f'API: OK — User: {d.get(\"name\",\"?\")} ({d.get(\"email\",\"?\")})')
except:
    print('API: FAILED')
" 2>/dev/null || echo "API not reachable"
```

### 4. Proxy Status

```bash
# Local proxy
curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:8787/v1/me 2>/dev/null | grep -q 200 && echo "Local proxy: UP" || echo "Local proxy: DOWN"

# FRIDAY proxy
curl -s -o /dev/null -w '%{http_code}' http://friday-mac-local:8787/v1/me 2>/dev/null | grep -q 200 && echo "FRIDAY proxy: UP" || echo "FRIDAY proxy: DOWN"
```

### 5. Recent Activity

```bash
bee conversations list --limit 1 --json 2>/dev/null | python3 -c "
import json,sys
try:
    d=json.load(sys.stdin)
    items = d if isinstance(d,list) else d.get('items',[])
    if items:
        c=items[0]
        print(f'Last conversation: {c.get(\"created_at\",\"?\")[:19]}')
        print(f'  Title: {c.get(\"title\",\"untitled\")[:60]}')
    else:
        print('No recent conversations')
except:
    print('Could not fetch conversations')
"
```

### 6. Device BLE Details (Reference)

If deeper debugging is needed, these are the known Bee BLE identifiers:

| Property | Value |
|---|---|
| Advertised Service | `0000d5c4-0000-1000-8000-00805f9b34fb` |
| Main Service | `03d5d5c4-a86c-11ee-9d89-8f2089a49e7e` |
| Control Char | `05e1f93c-d8d0-5ed8-dd88-379e4c1a3e3e` |
| Audio Char | `b189a505-a86c-11ee-a5fb-8f2089a49e7e` |
| SMP Service | `8d53dc1d-1db7-4cd3-868b-8a527460aa84` |
| Manufacturer ID | `0xBEEE` |

### 7. Daily brief (/bee-brief)

When generating the Bee daily brief (calendar, email, todos, recent conversations):

- **If brief is empty:** Bee may not be logged in. Run step 1 (CLI Status) and step 3 (API Access). If "Not logged in" or API fails, tell the user to run `bee login` and re-run the brief.
- **Large `bee now` output:** `bee now --json` can be very large (transcriptions). If the response is written to a file and parsing fails (truncated JSON), extract only what the brief needs instead of loading the full file:
  ```bash
  bee now --json 2>/dev/null | jq -c '.conversations[] | {id, start_time, end_time, short_summary, summary}' 2>/dev/null
  ```
  Use that stream (or first N lines) to list conversations with duration and summary without parsing the full payload.

### 8. Known Issues

- **Stuck blue light:** Stale BLE bond. Try 5 rapid button presses, or iOS Settings > General > Reset Network Settings.
- **Weak RSSI (-80+):** Low battery or antenna blocked. Charge via USB-C.
- **Encryption insufficient:** Device bonded to another device. Clear bonds on both sides.

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

