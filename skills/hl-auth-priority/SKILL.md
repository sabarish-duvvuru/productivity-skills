---
name: Headless Auth Priority
description: Use when resolving credentials through a priority chain instead of relying on a single auth source.
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-auth-priority
  icon_style: craft-category-glyph-v1
---

# Auth Priority

**Trigger**: Credential management, "wrong API key", auth failures

**Purpose**: Manage credentials with priority fallbacks instead of single source.

## Pattern

**Problem**: Hardcoded auth source breaks when credentials rotate or move.

**Solution**: Priority chain (env var → keychain → secrets manager → config file → fail).

## Implementation

### Atomic Auth Function

```python
import os
import subprocess
import json
from pathlib import Path

def get_credential(service, env_var=None, keychain_entry=None, config_path=None):
    """Get credential with priority fallback."""
    
    # Priority 1: Environment variable (immediate, no disk I/O)
    if env_var and (cred := os.environ.get(env_var)):
        return cred
    
    # Priority 2: Keychain (secure, OS-managed)
    if keychain_entry:
        try:
            result = subprocess.run(
                ["security", "find-generic-password", 
                 "-s", keychain_entry, "-w"],
                capture_output=True, text=True, timeout=5
            )
            if result.returncode == 0 and result.stdout.strip():
                return result.stdout.strip()
        except (subprocess.TimeoutExpired, FileNotFoundError):
            pass
    
    # Priority 3: Secrets manager (bw, 1pass, etc.)
    try:
        result = subprocess.run(
            ["bw", "get", "password", service],
            capture_output=True, text=True, timeout=10
        )
        if result.returncode == 0:
            return result.stdout.strip()
    except (subprocess.TimeoutExpired, FileNotFoundError):
        pass
    
    # Priority 4: Config file (fallback, plaintext)
    if config_path and (path := Path(config_path)).exists():
        try:
            data = json.loads(path.read_text())
            return data.get("api_key") or data.get("credential")
        except (json.JSONDecodeError, KeyError):
            pass
    
    raise RuntimeError(f"No valid credential found for {service}")

# Usage
api_key = get_credential(
    service="zai-api",
    env_var="ZAI_API_KEY", 
    keychain_entry="opencode",
    config_path="~/.config/service/credentials.json"
)
```

### Auth Testing Script

```python
def test_auth_sources(service):
    """Test all auth sources and report what works."""
    sources = {
        "env_var": os.environ.get("API_KEY"),
        "keychain": subprocess.run(
            ["security", "find-generic-password", "-s", service, "-w"],
            capture_output=True, text=True
        ).stdout.strip() if Path("/usr/bin/security").exists() else None,
        "config": Path(f"~/.config/{service}/credentials.json").expanduser()
    }
    
    working = {k: v for k, v in sources.items() if v}
    return {
        "service": service,
        "available_sources": list(working.keys()),
        "recommended": "env_var" if "env_var" in working else next(iter(working))
    }
```

### Credential Rotation Pattern

```python
def rotate_credentials(service, old_key, new_key):
    """Rotate credential across all sources."""
    
    # Update env var (current session)
    os.environ[f"{service.upper()}_API_KEY"] = new_key
    
    # Update keychain
    subprocess.run([
        "security", "add-generic-password",
        "-a", new_key, "-s", service, "-w"
    ])
    
    # Update config file
    config_path = Path(f"~/.config/{service}/credentials.json").expanduser()
    config_path.parent.mkdir(parents=True, exist_ok=True)
    config_path.write_text(json.dumps({"api_key": new_key}))
    
    # Invalidate old key
    subprocess.run(["security", "delete-generic-password", "-s", old_key])
```

## Rules

- **Env vars always win** - they're immediate and process-scoped
- **Keychain second** - OS-managed, survives reboots
- **Secrets managers third** - bw, 1pass, vault agents
- **Config files last** - plaintext, only for local tools
- **Never hardcode** credentials in scripts
- **Test all sources** before relying on one
- **Document priority** in skill/hook comments

**Crystallizes from**: Z.AI auth priority fix 2026-04-20 - env var worked, keychain was stale pool.

**Atomic output**: Single credential function with fallback chain + rotation support.


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

