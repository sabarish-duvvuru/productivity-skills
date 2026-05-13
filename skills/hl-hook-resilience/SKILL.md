---
name: Headless Hook Resilience
description: 'Use when the user needs: Make hooks resilient to missing volumes, networks, and external dependencies.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-hook-resilience
  icon_style: craft-category-glyph-v1
---

# Hook Resilience

**Trigger**: Hook failures, missing external resources, "hook not working"

**Purpose**: Make hooks resilient to missing volumes, networks, or external dependencies.

## Pattern

**Problem**: Hooks crash when external resources unavailable (SHADOW drive, network, APIs).

**Solution**: Existence checks + graceful exits + error logging.

## Implementation

### 1. Resource Existence Check

```python
from pathlib import Path

SHADOW_ROOT = Path("/Volumes/SHADOW")
if not SHADOW_ROOT.exists():
    sys.exit(0)  # Clean exit, not error
```

### 2. Write Permission Test

```python
# Test write permission before attempting I/O
try:
    test_file = SHADOW_ROOT / ".shadow-write-test"
    test_file.touch()
    test_file.unlink()
except (OSError, PermissionError):
    sys.exit(0)  # Clean exit
```

### 3. Network API Fallback

```python
endpoints_to_try = [primary_url, fallback_url]
for api_url in endpoints_to_try:
    try:
        result = call_api(api_url)
        return result  # Success
    except (Timeout, ConnectionError):
        continue  # Try next
return default_value  # All failed
```

### 4. Authentication Priority

```python
def get_credential():
    # Priority 1: Environment variable
    if cred := os.environ.get("API_KEY"):
        return cred
    
    # Priority 2: Keychain/secrets manager  
    try:
        return keychain_get("service-name")
    except Exception:
        pass
    
    # Priority 3: Config file
    return read_config("credential")
```

## Atomic Hook Template

```python
#!/usr/bin/env python3
"""Resilient hook template."""

import sys
from pathlib import Path

def check_prerequisites():
    """Exit cleanly if prerequisites missing."""
    required_paths = [
        Path("/Volumes/SHADOW"),
        Path.home() / ".Codex" / "config.json"
    ]
    
    for path in required_paths:
        if not path.exists():
            sys.exit(0)
    
    # Test write permission
    try:
        (Path.home() / ".Codex" / ".write-test").touch()
        (Path.home() / ".Codex" / ".write-test").unlink()
    except OSError:
        sys.exit(0)

def main():
    check_prerequisites()
    # Hook logic here
    return output

if __name__ == "__main__":
    main()
```

## Rules

- **Always exit 0** when external resource missing, not error codes
- **Log degradation** to stderr for debugging
- **Provide defaults** when primary unavailable
- **Test offline scenarios** before deploying

**Crystallizes from**: hook error fixes 2026-04-20 + SHADOW drive resilience patterns


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

