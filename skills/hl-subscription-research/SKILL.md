---
name: 'Headless Subscription Research'
description: 'Use when the user needs: Research and optimize AI/ML subscriptions with fresh data verification, portfolio analysis, and automated tracking'
icon: icon.svg
triggers:
- User asks about subscriptions
- User asks about pricing
- User asks about model availability
- User wants to optimize subscription portfolio
- What subscriptions do I have?
- Review my subscriptions
- Optimize my AI subscriptions
metadata:
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-subscription-research
  icon_style: craft-category-glyph-v1
---

# Subscription Research & Optimization

## When to Use This Skill

Use when user asks:
- "What subscriptions do I have?"
- "Review my subscriptions"
- "Optimize my AI subscriptions"
- "Pricing for [service]"
- "What models are available?"
- "Cancel subscription X"

---

## Critical Rules

### 1. Verify Freshness First
```
BEFORE using any cached subscription data:
- Check if data is >1 day old → RE-VERIFY
- Check if data is from another session → RE-VERIFY
- Pricing, models, auth = CRITICAL data
- Default: Ask user to confirm current status
```

### 2. User Confirmation > All Sources
```
Verification priority:
1. User tells you directly → TRUST
2. User confirms from official source → TRUST
3. Official website → MAY NEED curl/browser (JS rendering issues)
4. Old metadata → STALE, don't use
```

### 3. Mark Unverified Data Clearly
```
If you can't verify:
- Mark as ❓ UNVERIFIED
- Don't present as fact
- Say "according to old metadata" or "needs verification"
```

---

## Approach

### Step 1: Check Existing Data
```bash
ls -la /Volumes/☯Duality/ShadowArchive/80-reports/ | grep subscription
head -5 signal-subscription-metadata-*.json
```

### Step 2: Verify with User
```
Ask user directly:
- "What's your monthly out-of-pocket?"
- "Which subscriptions are active?"
- "Any prepaid/promotional subscriptions?"
- "When do renewals expire?"
```

### Step 3: Cross-Reference Official Sources (Optional)
```
Only if user requests verification:
- Use browser tool for JS-rendered sites
- curl often blocked by Cloudflare
- CLI tools (~/bin/zai quota) work well
```

### Step 4: Analyze Portfolio
```
Classify subscriptions:
- Operator Seats (human-facing, not broker-routable)
- Execution Lanes (broker-routable for automation)

Determine value:
- Monthly recurring cost
- Prepaid sunk costs (extract value)
- Free/promotional (use while available)
```

### Step 5: Provide Recommendations
```
For each subscription:
- Keep (actively used, good value)
- Evaluate (overlap with others, needs testing)
- Cancel (unused, poor value)
```

---

## Portfolio Strategy (SHADOW Framework)

### Operator Seats (Human-Facing)
- ChatGPT Pro $200 → Premium operator lane
- Cursor Pro → IDE-native workflow
- Gemini AI Pro → Google workflow lane

### Execution Lanes (Broker-Routable)
- Z.ai / GLM → Cloud burst (prepaid credits)
- OpenRouter → Fallback (pay-per-use)
- Local SHADOW → Private lane (free)

### Routing Logic
- Premium review → ChatGPT Pro
- Coding/IDE → Cursor Pro
- Google workflow → Gemini Pro
- Cloud burst → Z.ai / GLM
- Model testing → OpenRouter
- Private/local → Local SHADOW

---

## Common Patterns

### Pattern 1: Prepaid vs Monthly
```
User says: "I paid $192 for a year"
Response: This is sunk cost. Use it aggressively to extract value.
```

### Pattern 2: Promotional Free
```
User says: "Free with Pixel until August"
Response: Mark as $0/month. Set reminder 30 days before expiry.
```

### Pattern 3: Overlap Detection
```
Cursor Pro vs ChatGPT Pro:
- Both have similar coding features
- If both at $20/mo → ask which workflow is preferred
```

---

## Files Referenced

```
/Volumes/☯Duality/ShadowArchive/80-reports/
├── signal-subscription-metadata-*.json
├── subscription-maximize-status-*.md
└── subscription-research-*.md

/Volumes/☯Duality/ShadowArchive/03-agent-roots/subscription-maximize/
├── subscription-analyzer.py
├── cost-tracker.py
├── task-router.py
└── swarm-orchestrator.py
```

---

## Automation (Post-Research)

```bash
cd /Volumes/☯Duality/ShadowArchive/03-agent-roots/automation
python3 auto-router.py "Review the code"
bash install-cron-fixed.sh
```

---

## What We Learned

### Problem Areas
1. Stale metadata presented as current
2. Simulated results from previous sessions
3. Future models presented as existing
4. Web scraping failures (Cloudflare, JS rendering)

### Solutions
1. Per-session verification
2. User confirmation first
3. Mark unverified data clearly
4. Deploy daily freshness checks

---

## Output Format

```markdown
## Subscription Status (VERIFIED FACTS ONLY)

**Date**: [Current date]
**Source**: User-verified

### Monthly Out-of-Pocket
- Total: $X/month

### Active Subscriptions
| Service | Cost | Status | Notes |

### Recommendations
- Keep / Evaluate / Cancel
```

---

**Version**: 1.0  
**Created**: April 26, 2026  
**Crystallize**: Re-verify critical data every 7 days
