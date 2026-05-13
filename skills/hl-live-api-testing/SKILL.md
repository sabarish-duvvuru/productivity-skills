---
name: Headless Live API Testing
description: 'Use when the user needs: Reverse engineer API behavior through live endpoint testing instead of trusting stale docs.'
icon: icon.svg
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-live-api-testing
  icon_style: craft-category-glyph-v1
---

# Live API Testing

**Trigger**: API integration, provider registry entry, "API not working"

**Purpose**: Reverse engineer API behavior through live testing instead of trusting documentation.

## Pattern

**Problem**: Documentation outdated or incomplete. Registry claims wrong subscription model, endpoints, or auth.

**Solution**: Test API live, observe actual behavior, document reality.

## Methodology

### 1. Endpoint Discovery

```bash
# Test documented endpoint
curl -X POST "https://api.example.com/v1/chat" \
  -H "Authorization: Bearer $KEY" \
  -d '{"model":"test","messages":[...]}'

# Test alternative endpoints
curl -X POST "https://api.example.com/coding/v1/chat" ...
curl -X POST "https://api.example.com/api/v1/chat" ...
```

### 2. Response Format Analysis

```python
import json

response = json.loads(curl_output)

# Document actual structure
print("Fields:", response.keys())
print("Has reasoning:", "reasoning_content" in response)
print("Error format:", response.get("error", {}).keys())
```

### 3. Error Message Semantics

```python
# Capture actual error meanings
if "balance" in str(error).lower():
    meaning = "Prepaid credits, not subscription"
elif "quota" in str(error).lower():
    meaning = "Rate limit or subscription tier"
elif "unauthorized" in str(error).lower():
    meaning = "Auth method wrong (key vs token vs oauth)"
```

### 4. Authentication Testing

```bash
# Test env var vs keychain vs secrets
export KEY="env-key"
curl -H "Authorization: Bearer $KEY" ...

# Test keychain
security find-generic-password -s service -w | \
  curl -H "Authorization: Bearer $(cat)" ...

# Test secrets manager
bw get password api-key | curl -H "Authorization: Bearer $(cat)" ...
```

## Atomic Testing Script

```python
#!/usr/bin/env python3
"""Live API tester template."""

import subprocess
import json
import os

def test_endpoint(endpoint, payload, auth_methods):
    """Test endpoint with multiple auth methods."""
    for auth_type, auth_value in auth_methods.items():
        try:
            result = subprocess.run([
                "curl", "-s", "-X", "POST", endpoint,
                "-H", f"Authorization: {auth_type} {auth_value}",
                "-H", "Content-Type: application/json",
                "-d", json.dumps(payload)
            ], capture_output=True, text=True, timeout=30)
            
            if result.returncode == 0:
                response = json.loads(result.stdout)
                return {
                    "endpoint": endpoint,
                    "auth_type": auth_type,
                    "status": "success",
                    "response_keys": list(response.keys()),
                    "has_error": "error" in response
                }
        except Exception as e:
            return {
                "endpoint": endpoint,
                "auth_type": auth_type,
                "status": "failed",
                "error": str(e)
            }
    return None

# Test suite
endpoints = [
    "https://api.provider.com/v1/chat",
    "https://api.provider.com/coding/v1/chat", 
    "https://api.provider.com/api/v1/chat"
]

auth_methods = {
    "Bearer": os.environ.get("API_KEY"),
    "Basic": base64_encode(f"{user}:{pass}"),
}

results = [test_endpoint(ep, test_payload, auth_methods) for ep in endpoints]
print(json.dumps(results, indent=2))
```

## Registry Documentation

### Before Documentation (Actual)

```json
{
  "providerId": "z-ai-subscription",
  "costKind": "subscription", 
  "bandwidthClass": "quota_window",
  "fiveHourPromptBudget": 2400
}
```

### After Live Testing (Reality)

```json
{
  "providerId": "z-ai-subscription",
  "costKind": "prepaid_credits",
  "bandwidthClass": "credit_balance",
  "authentication": {
    "primary": "$ZAI_API_KEY environment variable",
    "note": "Balance errors mean recharge needed"
  }
}
```

## Rules

- **Trust curl output over docs**
- **Test multiple endpoints** (docs often omit specialist endpoints)
- **Document actual error messages** (not assumed meanings)
- **Test auth methods** (env var vs keychain differences matter)
- **Update registry from evidence** (not provider claims)

**Crystallizes from**: Z.AI integration 2026-04-20 - subscription model completely wrong, fixed by live testing.

**Atomic output**: List of working endpoints + auth methods + actual error semantics.


## Pipeline

```
Intent → Resolve target → Execute → Validate → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`

## Contract

- Destructive operations require explicit operator confirmation
- Always dry-run before applying changes

