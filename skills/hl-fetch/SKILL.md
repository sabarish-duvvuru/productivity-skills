---
name: Headless Fetch
description: 'Use when the user needs: Headless general-purpose web fetcher — extract content from arbitrary URLs (blogs, papers, docs, APIs, forums, archives) via firecrawl primary, curl+jq for API endpoints, agent-browser fallback for JS-heavy or auth-gated pages. Use when the user asks to "fetch this page", "read this URL", "extract from", "pull content from", "scrape", "get article", "download content", "check this link", or when shadow-research needs web content ingestion. Do NOT use for HN/X/YouTube/Git'
icon: icon.svg
allowed-tools: Bash(firecrawl:*), Bash(agent-browser:*), Bash(npx agent-browser:*), Bash(curl:*), Bash(python3:*), Read, Write
metadata:
  shadow_family: research
  shadow_role: executor
  shadow_scope: atomic
  capability_summary: URL-agnostic web content extraction via firecrawl + fallbacks
  openclaw_surfaces:
  - codex
  category: automation/headless
  family: hl
  lifecycle: active
  canonical_slug: hl-fetch
  icon_style: craft-category-glyph-v1
  invisible_first: true
  tier: 2
---

# Headless Web Fetch

General-purpose content extraction from any URL. Feeds into shadow-feed lanes and shadow-research pipeline.

## Source Routing

Before fetching, check if a specialized skill should handle it:

| URL Pattern | Route To |
|-------------|----------|
| `news.ycombinator.com` | hl-hn |
| `x.com`, `twitter.com` | hl-x |
| `youtube.com` | hl-youtube |
| `github.com` (trending, notifications) | hl-github |
| Everything else | **hl-fetch** (this skill) |

## Method 1: Firecrawl (preferred — fast, structured)

Works for any public URL. Returns clean markdown optimized for LLM consumption.

```bash
# Blog post, article, docs page
firecrawl scrape https://example.com/blog/article

# Research paper (arxiv, papers with code, etc.)
firecrawl scrape https://arxiv.org/abs/2504.19874

# Documentation
firecrawl scrape https://docs.example.com/api/reference

# Multiple pages (crawl)
firecrawl crawl https://docs.example.com --limit 10
```

## Method 2: curl + jq (API endpoints)

When the URL returns JSON directly:

```bash
# Detect JSON response
curl -sI URL | grep -i "content-type.*json"

# Fetch and format
curl -s URL | python3 -m json.tool

# Extract specific fields
curl -s URL | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(json.dumps(data, indent=2))
"
```

## Method 3: Agent-Browser (fallback — JS-heavy or auth-gated)

Use only when firecrawl fails (paywalled, heavy JS rendering, login required):

```bash
agent-browser open URL --profile "Sabarish"
agent-browser snapshot -i
agent-browser screenshot /tmp/hl-fetch-screenshot.png

# Scroll for lazy-loaded content
agent-browser scroll down 3
agent-browser wait 2000
agent-browser snapshot -i
```

## Content Type Detection

Adapt extraction based on URL patterns:

| Type | URL Signals | Extraction Strategy |
|------|-------------|-------------------|
| **Paper** | arxiv.org, openreview.net, papers.* | Title, authors, abstract, key findings |
| **Blog** | /blog/, medium.com, substack | Title, author, date, full text |
| **Docs** | /docs/, /api/, /reference/ | Structure headings, code blocks, parameters |
| **Forum** | /discuss/, /forum/, reddit.com | Top posts, vote counts, key replies |
| **API** | /api/, JSON content-type | Schema, endpoints, example responses |
| **News** | /news/, press releases | Headline, date, key facts |

## Output Format

Write structured JSON intermediate to `/tmp/`:

```json
{
  "id": "fetch-{sha256_of_url_first8}",
  "url": "https://...",
  "title": "Extracted page title",
  "author": "Author if available",
  "date": "Publication date if available",
  "content_md": "Full markdown content",
  "content_type": "paper|blog|docs|forum|api|news|unknown",
  "domain_lane": "ai-ml|security|tools|engineering|product|general",
  "word_count": 1234,
  "extracted_at": "2026-03-25T14:30:00Z",
  "method": "firecrawl|curl|agent-browser"
}
```

Save to: `/tmp/hl-fetch-{hostname}-{timestamp}.json`

## shadow-feed Integration

After extraction, classify into lanes:

```bash
# Write JSON array wrapper and classify
python3 -c "
import json
with open('/tmp/hl-fetch-output.json') as f:
    item = json.load(f)
# Wrap as array for ingest.sh compatibility
with open('/tmp/hl-fetch-classify.json', 'w') as f:
    json.dump([item], f)
"

# Run lane classifier
bash /Volumes/SHADOW/ShadowArchive/10-projects/shadow-feed/ingest.sh --classify /tmp/hl-fetch-classify.json
```

## shadow-research Integration

If fetched content looks kernel-worthy (research papers on topics matching user's domain, technical docs for active projects), flag for the research pipeline:

```bash
# Append to kernel-diff queue for shadow-research Phase 1 crawl
python3 -c "
import json, uuid
from datetime import datetime, timezone

entry = {
    'id': str(uuid.uuid4()),
    'timestamp': datetime.now(timezone.utc).isoformat(),
    'source_node': 'jarvis',
    'hook': 'hl-fetch',
    'captures': {
        'url': 'FETCHED_URL',
        'content_type': 'DETECTED_TYPE',
        'content_summary': 'FIRST_500_CHARS',
        'domain_lane': 'CLASSIFIED_LANE',
        'training_pairs': []
    }
}

import os
queue_dir = os.path.expanduser('~/.shadow-research/queue')
os.makedirs(queue_dir, exist_ok=True)
with open(f'{queue_dir}/kernel-diff.jsonl', 'a') as f:
    f.write(json.dumps(entry) + '\n')
"
```

## Batch Fetch

For multiple URLs:

```bash
# Fetch multiple URLs in sequence
for url in URL1 URL2 URL3; do
  firecrawl scrape "$url"
done
```

## Daily Digest

Output path for daily research digests:

```
/Volumes/SHADOW/ShadowArchive/10-projects/shadow-feed/daily/fetch-YYYY-MM-DD.md
```

Format:
```markdown
# Fetch Daily — YYYY-MM-DD

## Papers
- [Title](url) — key finding summary

## Docs
- [Title](url) — relevant section

## Articles
- [Title](url) — author, key insight
```


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

