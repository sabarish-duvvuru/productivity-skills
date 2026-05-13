---
name: Operator Patterns
description: 'Use when the user needs: Learned interaction patterns specific to this operator. Apply automatically — never ask about these, just do them. Trigger on any session with this operator.'
icon: icon.svg
metadata:
  depends_on: [browser-surface-bridge,naming-conventions,wacli]
  category: workflow/process
  family: workflow
  lifecycle: active
  canonical_slug: operator-patterns
  icon_style: craft-category-glyph-v1
---

# Operator Interaction Patterns

Learned behaviors from working with this operator (Sabarish Duvvuru). Apply automatically without being asked. These are non-negotiable defaults.

## 0. Naming Conventions (Auto-Apply)

When naming or renaming **anything** — session, chat, file, branch, commit, email subject, calendar event, deliverable — apply **`naming-conventions`** skill automatically. Never ask what to name something. Follow the rules.

Key limits:
- M365 Copilot Chat / Craft session: **≤50 chars**
- Email subject: **≤80 chars** practical
- Git commit subject: **≤50 chars** ideal
- Always prefix with program code (FH4S, XP7G, etc.)
- Always append date as YYMMDD
- Use abbreviation table from the skill
- Run the 7-point validation check after naming

## 1. Narrate Intent, Not Tool Names

When telling the operator what you're about to do, describe the **destination**, not the mechanism.

| Bad | Good |
|-----|------|
| "Listing Chrome pages" | "Checking what's open in Chrome" |
| "Calling chrome-devtools_list_pages" | "Checking open tabs" |
| "Using grep to search" | "Searching for X across the codebase" |
| "Invoking bash with git status" | "Checking git status" |
| "Navigating to x.com/i/bookmarks" | "Opening X bookmarks" |

**Rule:** The operator sees the *intent*, not the plumbing. Describe where we're going, not which tool gets us there.

## 2. Cross-Reference Catalog Before Guessing

When the operator mentions a name, email, account, or credential by voice or vague reference:

1. **Check AGENTS.md first** — the profile map, agent personas, and key people tables are authoritative
2. **Check keychain/bitwarden** — `security find-generic-password`, `bw get`
3. **Then confirm** — if still unsure, show the catalog match and ask

**Never transcribe a spoken email/name without checking the catalog.** The operator has a specific set of known identities. Misheard pronunciations resolve against the catalog, not against phonetic guessing.

### 2a. Explicit Correction Beats Catalog

If the operator explicitly corrects an identity, account, profile, or ownership boundary:

1. lock to the correction immediately
2. do not let old AGENTS tables, memory, browser history, or recent tool output override it
3. restate the corrected lock before acting when the task is sensitive to that identity

Example:
- If the operator says Google Recorder is always on `sabarish.duvvuru@gmail.com` main profile, do not switch to `sduvvuru` or any other profile later in the same task because another surface suggests it.

### Known Identity Map (from AGENTS.md)

**Chrome Profiles:**

| Dir | Username | Email | Space |
|-----|----------|-------|-------|
| Default | sdluffy | sabarish.duvvuru@gmail.com | Private (`Sabbu`) |
| Profile 1 | Mac | clawd.ctrl@gmail.com | Agent |
| Profile 2 | ishuru | ishuru.dev@gmail.com | Personal |
| Profile 5 | sduvvuru | dsabarish.work@gmail.com | Professional |

**Agent Personas:**

| ID | Name | Email | Purpose |
|----|------|-------|---------|
| mac | Mac Clawd | clawd.ctrl@gmail.com | Agent persona |
| leo | Leo (Leonardo Stark) | theleostark@outlook.com | Agent persona |
| red | Red | sabarish.duvvuru.qx@astemo.com | Astemo agent |

**Operator Aliases:** `sdluffy` (macOS), `sabbu` (nickname), `sabarish` (first name)

## 3. Reference Chrome Profiles by Username

Always use the **username** when identifying or discussing Chrome profiles. Never use directory paths.

| Say | Never say |
|-----|-----------|
| "Opening on ishuru's profile" | "Using isolatedContext=Profile 2" |
| "sduvvuru's Chrome" | "Profile 5" |
| "Mac's profile" | "Profile 1" |
| "sdluffy's Chrome" | "Default profile" |

## 3a. Browser Profile Boundary Rules

Treat browser profile routing as an identity boundary, not a convenience setting.

- `sdluffy` / Default is personal and should display as `Sabbu`.
- `sduvvuru` / Profile 5 is Professional and is the only place for Astemo M365, SharePoint, Teams, Outlook, Smartsheet, and work tab groups.
- Work URL markers are `astemogroup.sharepoint.com`, `astemogroup-my.sharepoint.com`, `outlook.office.com`, `teams.microsoft.com`, and `smartsheet.com`.
- Personal URL/task markers are government, finance, medical, taxes, car/DMV/BMV, personal Google Calendar/Drive/Keep, leases, insurance, and identity documents. These belong in `sdluffy` / `Sabbu`, never in the work profile.
- **ChatGPT URL markers are `chatgpt.com` and `chat.openai.com`: always use the main `sdluffy` / `Sabbu` Chrome profile logged in as `sabarish.duvvuru@gmail.com`. Do not use Craft's in-app Chromium for ChatGPT; it consistently hits the Cloudflare/security verification wall.**
- Before saving anything durable in an authenticated app (Calendar event, reminder, note, form draft), verify the visible account/profile and destination calendar/workspace. If the profile or calendar is ambiguous, stop and ask; do not rely on the browser's existing session state.
- If a work URL is open in `sdluffy`, migrate it to `sduvvuru` by opening the same URL in the work profile and closing the personal-profile tab/group.
- If a personal/government/finance URL or reminder is open in `sduvvuru`, stop and redirect to `sdluffy` before continuing.
- Saved/pinned tab groups follow the same rule: work groups only in `sduvvuru`; personal groups only in `sdluffy`; dev/open-source groups in `ishuru`.
- Before scripted M365/SharePoint/Teams work, run the profile drift checker when there is any doubt:
  `python3 ~/.agents/skills/hl-automation-framework/scripts/chrome_profile_drift_check.py`

### 3b. Sensitive Personal Transaction Gate

For BMV/DMV, tax, immigration, banking, medical, insurance, or other identity/payment flows:

1. Open the official site only after confirming the personal `sdluffy` / `Sabbu` profile.
2. Prefill only low-sensitivity fields already stated in chat (for example DOB or public reminder text).
3. Do not extract, display, or paste high-sensitivity identifiers from Drive/Notion/Keep into forms: driver license number, SSN/I-94 last 4, passport/A-number, payment card, bank data, full tax identifiers.
4. Let the operator type high-sensitivity fields directly. After login, assist with non-sensitive review, navigation, and decision support.
5. Stop before payment or final government/legal submission and request explicit confirmation.

If the operator says "I approve" or "do it," that authorizes navigation/review, not automated extraction or entry of high-sensitivity identifiers.

## 4. Voice Mishearing Recovery

When the operator says something that sounds like a name/email/URL but doesn't match known catalog entries:

1. Check AGENTS.md identity tables for phonetic matches
2. If close match found, confirm: "You mean clawd.ctrl@gmail.com, right?" (not "Did you say cloud dot control?")
3. If no match, ask directly with the catalog context: "I couldn't find that in the known accounts. Which profile?"

**Key mishearing examples:**
- "cloud.control" → catalog says `clawd.ctrl@gmail.com` (Mac Clawd)
- "okamotosam" → catalog says Atsushi Okamoto
- "Силкретор, Тул Креато" → not Russian — catalog says skill-creator, tool-creator (existing skills)

**General rule:** When something sounds foreign or unfamiliar, check the skill/tool name inventory before assuming a language difference. The operator uses `~/.agents/skills/` and `~/bin/` — search those first.

## 5. Thinking Block Preference

The operator prefers minimal intermediate output. Show final results, not reasoning chains. Don't burn permission prompts on repeated failed attempts — stop and ask after the first failure.

### 5b. Resolve Source References Before Asset Retrieval

When the operator asks to get, install, summarize, extract, or use something **from** a named person/channel/video/article/tweet/email/chat/recent item, treat the source phrase as a hard provenance constraint, not a search hint.

Examples:
- "get the skill from Riley" + a YouTube Recent mention means: resolve Riley through recent YouTube history first, then extract the linked skill.
- "use Bryan's email" means: open/resolve the specific email thread first, not a generic file or web search.
- "from that video/tweet/chat" means: recover the referenced source before acting on the asset.

Default workflow:
1. Parse as `asset X` from `source Y`.
2. Resolve `source Y` using the active context surface first: mentioned skill, last widget, browser/history, email/thread, file, or chat context.
3. Extract links, description, transcript, attachments, or claims from that source.
4. Only then search globally, and only as fallback or verification.
5. If the source cannot be resolved, stop and say what was unresolved; do not silently substitute a plausible public result.

Failure mode to avoid: over-weighting the noun (`excalidraw-diagram-skill`) and dropping the provenance (`from Riley`).

## 5a. ChatGPT Latest Chat Retrieval

When the operator asks for “latest chat,” “get latest chat,” or similar ChatGPT session recovery:

1. Default to **ChatGPT web**, not Teams or M365, unless the operator explicitly says otherwise.
2. Do **not** rely on the in-app Craft browser for ChatGPT because Cloudflare/security verification blocks it.
3. Use the operator’s already logged-in **sdluffy / Sabbu Chrome** session — main Chrome profile, account `sabarish.duvvuru@gmail.com` — for all ChatGPT retrieval and organization work.
4. “Latest” means **most recent across all ChatGPT chats**, including pinned chats. Do **not** skip pinned chats just because they are pinned; pinned chats can be the latest active work.
5. Primary recency source is Chrome History, not ChatGPT sidebar order:
   - Run `chatgpt-recent` first. It reads Chrome History across profiles and sorts `chatgpt.com/c/*` / `chat.openai.com/c/*` by `last_visit_time`.
   - Prefer `Default` / **sdluffy** for ChatGPT unless the operator explicitly names another profile.
   - Ignore URL fragments such as `#settings`, `#pricing`, etc. Canonicalize to the `/c/<id>` URL before comparing or opening.
   - Deduplicate by conversation id (`/c/<id>`). Keep the newest timestamp per id.
6. Treat `cg-cli chatgpt-recent` / `chatgpt-cli chatgpt-recent` as a **sidebar scraper**, not the authority for “latest.” Its documented behavior is “first non-pinned”; that is useful only as fallback when history is unavailable.
7. Candidate selection order:
   1. Current/front Chrome tab if it is a ChatGPT conversation the operator is actively using.
   2. Newest canonical `/c/<id>` from `chatgpt-recent` in sdluffy profile.
   3. If sdluffy has no candidate, list candidates from other profiles with profile names.
   4. Sidebar scrape only if Chrome History fails or is stale.
8. If several candidate chats are close or ambiguous, list title + profile + URL + last visited time and ask the operator to pick; do not silently choose first non-pinned.
9. After selecting the most recent candidate, open that canonical URL in logged-in sdluffy Chrome and pull the transcript.
10. If CLI execution is blocked by permission mode, ask once to switch from Explore to Ask/Allow All; do not keep trying browser workarounds.

## 6. Shared Queue Discipline (from AGENTS.md)

When writing to Reminders/Notes (syncs to iCloud, visible on company devices):
- Use direct, generic language
- Describe *what* needs to happen, not the technique
- Keep infrastructure knowledge local (AGENTS.md, skills, scripts on thumb drive)
- The shared queue carries the task, not the mechanism

## 7. Information Boundary

Pattern knowledge and infrastructure details must NEVER appear in company-visible surfaces (iCloud Reminders, Dropbox, Notes shared to work). The mechanism lives on the thumb drive; the task goes to the queue.

## 8. Trustworthy Artifact Completion

When creating guides, docs, reports, decks, SOPs, specs, diagrams, spreadsheets, HTML previews, research briefs, source-derived notes, or implementation plans for this operator:

1. Stop optimizing for fast “file created” completion. Optimize for: **the operator can trust this artifact immediately**.
2. Resolve deliverable type and audience before writing.
3. Extract source material thoroughly enough; for video/audio, use transcript plus timestamp structure and add visual/screenshot pass when UI, layout, or on-screen behavior matters.
4. Separate extraction from synthesis: source facts first, system mapping second, final artifact third.
5. Always render a useful preview directly in chat unless explicitly told not to. A saved path alone is not enough.
6. Before claiming completion, state bounded completion: evidence used, verification done, residual risk/gaps, and next best improvement.
7. Honor requested omissions and audience boundaries; never expose hidden runtime/core implementation details in docs when the operator says not to.

## 9. GLM Routing Habit

When the operator says "use GLM", "route to Z.AI", or "max out the plan":

1. **Run `glm-gate` first** — check the prompt/file context for RED-tier markers before sending
2. **Never send Astemo/FH4S/Honda content to GLM** — even if the operator says "just use GLM" — gate override requires explicit `--force`
3. **GREEN-tier tasks** — maximize GLM usage (vision, public coding, web synthesis, HTML prototypes)
4. **AMBER-tier tasks** — prefer Gemini first, GLM allowed with metadata stripped
5. **If blocked, explain why** — "GLM-001 blocks this because [RED marker] detected. Routing to [local/Gemini] instead."

## 10. Deletion Carefulness — Extract Before Deleting

When pruning worktrees, branches, sessions, files, or any batch of items:

1. **Inventory first** — list what each item contains (commit subjects, file changes, session names) before removing anything.
2. **Classify value** — tag each item as: keep (active work), salvage (valuable but orphaned), safe to delete (truly empty/superseded).
3. **Protect orphans** — save any orphan commits to named refs (`refs/salvage/<name>`) before pruning, so GC doesn't eat them.
4. **Extract unique value** — for salvage items, diff against current state to find what's genuinely new vs already present.
5. **Promote or drop** — cherry-pick or copy unique content into the working tree, then drop the ref.
6. **Only then delete** — prune the empty/confirmed-superseded items last.

**Never batch-delete first and salvage second.** The salvage step is not optional cleanup — it's the primary step. Deletion is the cleanup.

This applies to: git worktrees, git branches, session status cleanup, file removal, skill archival, and any bulk prune operation.

### Key heuristics
- If you haven't read what's inside, you haven't earned the right to delete it.
- "Detached" or "stale" does not mean "empty" — it means "unreachable from current view."
- A commit that took 2 hours of agent work takes 5 seconds to protect with a named ref.
- When in doubt, save the ref and note it in the audit report. The operator can drop it later.

## 11. Invisible-First Execution — Never Steal Operator Focus

Every tool invocation has a **visibility cost**. An agent popping a browser window, switching Chrome tabs, or foregrounding an app yanks the operator out of their flow. This is not a minor inconvenience — it breaks the entire reason for having an agent.

### The Rule

**Try the invisible path first. Only use visible browser automation as a last resort, and only when explicitly asked.**

### Tool Visibility Spectrum

| Tier | Method | Focus Impact | When to use |
|------|--------|-------------|-------------|
| 0a | CLI with cookie delegation (`yt-dlp --cookies-from-browser`, `gh`) | ✅ Invisible | **Always first for data extraction** |
| 0b | `cua-driver` MCP/CLI — AX dispatch, per-PID events, background screenshots | ✅ Invisible | **Always first for GUI automation** |
| 1 | CLI with API token (`bw`, `wacli`, `gcloud`, `tailscale`) | ✅ Invisible | **Second** |
| 2 | Cookie export → `curl -b cookies.txt` (no browser process) | ✅ Invisible | **Third — needs export script** |
| 3 | Safari AppleScript (`do JavaScript`, `get URL`) | ✅ Invisible | Focus-safe browser injection |
| 4 | Chrome AppleScript (`execute javascript`, `get title`) | ⚠️ May steal focus | Risk varies by macOS state — prefer cua-driver or Safari |
| 5 | `osascript activate` / `open -a` / `open location` | 🔴 ALWAYS foregrounds | **Banned without explicit ask** |
| 6 | `browser_tool --foreground` | 🔴 ALWAYS foregrounds | **Banned without explicit ask** |

### cua-driver — Background GUI Automation

`cua-driver` (v0.1.2) is installed at `/Users/sdluffy/.local/bin/cua-driver` with `CuaDriver.app` in `/Applications`. It provides **30+ tools** for driving macOS apps in the background:

| Capability | cua-driver tool | What it replaces |
|---|---|---|
| Launch app hidden | `launch_app({bundle_id})` | `open -a`, `osascript activate` |
| Screenshot window | `get_window_state({pid, window_id})` | `screencapture`, `screenshot` |
| Click by element | `click({pid, element_index})` | `cliclick`, `osascript click` |
| Click by pixel | `click({pid, x, y})` | `cliclick c:x,y` |
| Type text | `type_text({pid, text})` | `osascript keystroke` |
| Press keys per-PID | `press_key({pid, key})` | `osascript key code` |
| Hotkey combo | `hotkey({pid, keys: ["cmd","c"]})` | `osascript keystroke + key down/up` |
| Scroll | `scroll({pid, direction, amount})` | `osascript scroll` |
| Set field value | `set_value({pid, element_index, value})` | DOM manipulation |
| Read page text | `page({pid, action: "get_text"})` | `execute javascript document.body.innerText` |
| Query DOM | `page({pid, action: "query_dom"})` | `execute javascript querySelectorAll` |
| Run JS in page | `page({pid, action: "execute_javascript"})` | `osascript execute javascript` |
| List windows | `list_windows({pid})` | `osascript every window` |
| List apps | `list_apps` | `ps aux`, `pgrep` |

**Key invariant:** snapshot before AND after every action (`get_window_state` → act → `get_window_state`).

**Daemon must be running:** `open -n -g -a CuaDriver --args serve` (auto-starts with `launch_app`).

**Permissions required:** Accessibility ✅ + Screen Recording ✅ (both granted).

### Critical: Chrome vs Safari Focus Behavior

**Every** `tell application "Google Chrome"` AppleScript call steals focus — including read-only queries. Measured 2026-05-03:

| Pattern | Before | After | Stole Focus? |
|---------|--------|-------|-------------|
| `tell app "Chrome" to get title` | Craft Agents | **Google Chrome** | ✅ YES |
| `tell app "Chrome" to execute javascript` | Craft Agents | **Google Chrome** | ✅ YES |
| `tell app "Chrome" to set URL of tab` | Craft Agents | **Google Chrome** | ✅ YES |
| `tell app "Chrome" to open location` | Craft Agents | **Google Chrome** | ✅ YES |
| `tell app "Safari" to do JavaScript` | Craft Agents | Craft Agents | ❌ NO |
| `tell app "Safari" to get URL` | Craft Agents | Craft Agents | ❌ NO |

**This means `browser-surface-bridge` — designed to be invisible — actually steals focus every time** because it uses `tell application "Google Chrome" ... execute javascript`.

Future fix: export Chrome cookies to file → use curl with cookie jar (Tier 2.5). Or migrate M365 auth to Safari (Tier 3).

### Forbidden Without Explicit Ask

These actions **always steal focus** and are banned unless the operator says "show me" or "open it":

- **ANY** `osascript` with `tell application "Google Chrome"` — even `get title` steals focus
- `osascript -e 'tell application "Google Chrome" to open location ...'`
- `osascript -e 'tell application "Google Chrome" to activate'`
- `osascript -e 'tell application "Google Chrome" ... execute javascript ...'` (browser-surface-bridge included)
- `open -a "Google Chrome" <url>`
- `browser_tool open --foreground`
- Any `osascript` that calls `activate` on any app
- Any `open` command that launches or foregrounds a GUI app

**Focus-safe alternative:** Safari AppleScript (`do JavaScript`, `get URL`) does NOT steal focus. Use Safari as the browser injection target when the operator is logged into the target service in Safari.

### CLI-First Tooling Map

When a skill calls for browser automation, check this map first. If a CLI tool exists, **use it instead of the browser**.

| Service | CLI Tool | Cookie/Auth Method | Replaces Browser Skill |
|---------|----------|--------------------|-----------------------|
| YouTube history/transcripts | `yt-dlp` | `--cookies-from-browser chrome:Default` | yt-history-headless, yt-recent-insights Playwright |
| GitHub repos/issues/PRs | `gh` | OAuth token (built-in) | Any GitHub browser workflow |
| Bitwarden vault | `bw` | Session key | bitwarden-browser |
| WhatsApp | `wacli` | Paired session | Slack WhatsApp workflows |
| Tailscale mesh | `tailscale` | Node auth | tailscale-mesh browser paths |
| Google Cloud | `gcloud` | OAuth (built-in) | gcloud browser workflows |
| Dropbox | Dropbox API via curl/Python | OAuth token | hl-dropbox browser paths |
| X/Twitter | `bird` CLI or yt-dlp cookies | Cookie delegation | hl-x browser extraction |
| **SharePoint** | **`export-cookies.py` → `curl -b cookies.txt`** | **Chrome cookie decryption** | **astemo-sharepoint, sp-crawl, m365-search, browser-surface-bridge** |
| **Outlook/M365** | **`export-cookies.py` → `curl -b cookies.txt`** | **Chrome cookie decryption** | **astemo-outlook, astemo-teams** |

### Cookie Export Script (Tier 2 — Invisible M365 Access)

Installed at: `~/.agents/skills/browser-surface-bridge/scripts/export-cookies.py`

Decrypts Chrome cookies using Keychain + PBKDF2 + AES-128-CBC. Handles Chrome v24+ hash prefix.

```bash
# Export SharePoint cookies from work profile
python3 ~/.agents/skills/browser-surface-bridge/scripts/export-cookies.py \
  -p "Profile 5" -d sharepoint -o /tmp/sp-cookies.txt

# Query SharePoint REST API — invisible, ~2 seconds
curl -b /tmp/sp-cookies.txt \
  "https://astemogroup.sharepoint.com/sites/Hoe-Axle/_api/web/lists?\$select=Title,ItemCount&\$top=5" \
  -H "Accept: application/json;odata=nometadata"

# Export Outlook cookies
python3 ~/.agents/skills/browser-surface-bridge/scripts/export-cookies.py \
  -p "Profile 5" -d outlook -o /tmp/outlook-cookies.txt

# Export YouTube cookies (alternative to yt-dlp --cookies-from-browser)
python3 ~/.agents/skills/browser-surface-bridge/scripts/export-cookies.py \
  -p Default -d youtube.com -o /tmp/yt-cookies.txt
```

**Security:** Output file gets chmod 600. Delete after use — contains decrypted auth tokens.

### Anti-Pattern: The Focus Cascade

What happened during the 2026-05-03 YouTube session (do not repeat):

```
1. browser_tool open       → popped in-app window (low impact but unnecessary)
2. browser_tool navigate   → loaded YouTube (signed out, useless)
3. osascript → Chrome      → STOLE FOCUS, opened Chrome
4. osascript JS inject     → more Chrome activation
5. yt-dlp --cookies...     → should have been step 1, invisible, 3 seconds
```

The correct path was **only step 5**. Steps 1-4 were entirely wasted and cost the operator their focus.

### When Browser IS Needed

Some tasks genuinely require a browser — there is no CLI alternative:
- M365 apps (SharePoint, Teams, Outlook Web) with no API access
- Web apps behind SSO with no CLI
- Visual verification of a rendered page
- Form filling on sites with no API

In those cases:
1. Use `browser_tool open` (background) + `evaluate` — stay invisible
2. Use `osascript` JS injection into **existing tabs** without activating the app
3. Only foreground when the operator asks to see the result

### Chromium vs Chrome

The Craft Agent `browser_tool` uses an **in-app Chromium** (separate from system Chrome). It does **not** have the operator's Google/YouTube/M365 cookies. This means:
- `browser_tool navigate https://youtube.com` → always signed out
- `yt-dlp --cookies-from-browser chrome:Default` → uses real Chrome cookies, always works
- Do not attempt auth-gated workflows in `browser_tool` — use CLI tools or real Chrome cookie delegation instead

## Pipeline

```
Input (any interaction with the operator)
  ↓
Catalog Check (AGENTS.md identity tables, keychain, skill inventory)
  ↓
Pattern Match (which of Patterns 0–11 applies?)
  ↓
Apply (auto-execute without asking — these are non-negotiable defaults)
  ↓
Artifact Generation (only when patterns produce durable output)
  ├── Skill updates (correction → immediate skill edit)
  └── ShadowArchive/80-reports/ (if audit generates a report)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Silent auto-application of all patterns during any interaction | Every session with this operator — never ask, just apply |
| `diagnostic` | Show which patterns are active and which were triggered this session | `diagnostic operator patterns`, debugging behavior |
| `audit` | Check for pattern violations in recent agent behavior | `audit operator patterns`, after a friction incident |
| `update` | Apply an operator correction to the relevant pattern section | After operator corrects behavior |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Pattern updates (from corrections) | `~/.agents/skills/operator-patterns/SKILL.md` | Updated interaction defaults |
| Chrome profile drift check | `~/.agents/skills/hl-automation-framework/scripts/chrome_profile_drift_check.py` | Profile boundary verification |
| Naming convention references | `~/.agents/skills/naming-conventions/SKILL.md` | Naming rule source |

## Fallback Chain

1. **Primary:** Apply all patterns from this skill automatically during any interaction
2. **AGENTS.md not accessible:** Use hardcoded identity map from this skill (Section 2a); note that catalog is degraded
3. **Keychain unavailable:** Ask operator for identity confirmation rather than guessing; do not fall back to phonetic matching
4. **Chrome profile map stale:** Run drift checker script; if script unavailable, ask operator to confirm profile
5. **Last resort:** Apply the safest default (narrowest permission, most conservative profile routing, explicit confirmation for ambiguous identities)

## Prerequisites

- Access to `AGENTS.md` for identity tables and catalog
- macOS Keychain for credential lookup (`security` CLI)
- Chrome browser with known profiles (for profile routing)
- Safari browser available as focus-safe alternative (for AppleScript injection)
- `naming-conventions` skill installed (for Pattern 0)
- CLI tools available: `yt-dlp`, `gh`, `bw`, `wacli`, `tailscale`, `gcloud` (for invisible-first execution)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Operator name/email doesn't match catalog | Ask with catalog context; do not phonetic-guess |
| Chrome profile ambiguous | Ask operator to confirm; do not assume based on URL alone |
| Keychain query fails for a known identity | Report; ask operator; do not fall back to stored plaintext |
| CLI tool not installed (invisible-first) | Report tool missing; suggest install command; fall back to next visibility tier |
| Browser focus stolen during invisible attempt | Apologize; note the tool behavior; suggest CLI upgrade path |
| Operator correction contradicts pattern | Correction wins immediately; update this skill; note in Crystallized From |

## Contract

- **Auto-apply, never ask.** These patterns are non-negotiable defaults learned from this operator. Apply them silently.
- **Operator correction is live truth.** When the operator corrects an identity, profile, path, or boundary, lock to the correction immediately. Update this skill before continuing.
- **Invisible-first execution.** Always try the invisible path (CLI with cookie delegation, API token, keychain) before any browser automation that steals focus.
- **Focus theft is banned** unless the operator explicitly says "show me" or "open it." Chrome AppleScript always steals focus. Safari AppleScript does not.
- **Information boundary.** Pattern knowledge and infrastructure details must NEVER appear in company-visible surfaces (iCloud Reminders, Dropbox, Notes shared to work). The mechanism lives on the thumb drive.
- **Deletion carefulness.** Extract before deleting. Named refs are cheap insurance. If you haven't read what's inside, you haven't earned the right to delete it.
- **Externalization rule.** Pattern corrections update this skill file. Never leave a correction only in chat memory.
- **Do not** use Chrome AppleScript for any read operation — it always steals focus. Use Safari instead.
- **Do not** route personal/government/finance tasks to the work profile. Ever.
- **Do not** use Craft's in-app Chromium for `chatgpt.com` / `chat.openai.com`; use main `sdluffy` / `Sabbu` Chrome logged in as `sabarish.duvvuru@gmail.com` because Craft Chromium hits Cloudflare/security verification.
- **Do not** enter high-sensitivity identifiers (SSN, DL#, payment cards) into forms, even with operator approval. The operator types those.

## Crystallized From

- Session: 2026-04-16 (JARVIS, OpenCode)
- Trigger: Operator corrected four behaviors across the session
  - Narrating tool names instead of intent
  - Transcribing "cloud.control" without checking AGENTS.md catalog (should be clawd.ctrl)
  - Using directory paths instead of usernames for Chrome profiles
  - Assuming "Силкретор, Тул Креато" was Russian — actually "skill creator, tool creator"
- Original task: Finding X.com bookmarks, then crystallizing session learnings
- Pattern: All corrections share the same root cause — the agent not using available context (AGENTS.md, skill inventory, identity map) before acting

- Session: 2026-04-30 (Craft Agents, BMV + Calendar)
- Trigger: Operator asked why a personal BMV/calendar task opened in the work profile and then invoked `--crystallize`.
- Original task: Find Indiana vehicle registration docs, create yearly Google Calendar reminder, and start myBMV renewal.
- Pattern: Authenticated browser state is not neutral. Personal government/vehicle/finance tasks require an explicit personal-profile/account/calendar verification gate before durable writes or form work. Approval does not override high-sensitivity identifier entry boundaries.

- Session: 2026-05-02 (Craft Agents, Shadow Agents guide)
- Trigger: Operator challenged half-hearted artifact completion after a YouTube-derived guide was saved without full preview/source mapping.
- Original task: Create an explanatory guide for Shadow Agents from a video, with hidden implementation/runtime details omitted.
- Pattern: Artifact requests require trust-grade completion: thorough source extraction, source/synthesis separation, in-chat preview, bounded completion evidence, and honest residual gaps before claiming done.

- Session: 2026-05-03 (Craft Agents, Duality workspace cleanup)
- Trigger: Operator asked "are you internalizing those or just deleting?" after 62 worktrees and 9 branches were pruned without value extraction.
- Original task: Workspace session cleanup + diamond slope worktree/branch organization.
- Pattern: Batch deletion must follow extract-then-delete order, not delete-then-salvage. Named refs are cheap insurance. "Detached" ≠ "empty." If you haven't read what's inside, you haven't earned the right to delete it.

- Session: 2026-05-03 (Craft Agents, YouTube skill surgery)
- Trigger: Operator asked "What did you learn from yt-dlp externalization?" after the agent wasted time opening browser windows (browser_tool, osascript Chrome activation) when yt-dlp --cookies-from-browser was the 3-second invisible path all along.
- Original task: Run yt-history-headless skill to extract YouTube watch history.
- Pattern: CLI tools with cookie delegation (yt-dlp, gh, bw, wacli) are always tier 0 — invisible, fast, no focus theft. Browser automation (osascript activate, browser_tool foreground, open -a) is tier 5-6 — banned unless operator explicitly asks to see it. The Craft Agent Chromium is always signed out; never attempt auth-gated workflows there. The invisible-first tooling map covers YouTube, GitHub, Bitwarden, WhatsApp, Tailscale, gcloud, Dropbox, and X/Twitter.

- Session: 2026-05-05 (Craft Agents, ChatGPT organization)
- Trigger: Operator corrected the agent after it tried `chatgpt.com` in Craft's in-app Chromium and hit Cloudflare/security verification again.
- Original task: Organize ChatGPT chats while skipping the active Photo project folder.
- Pattern: `chatgpt.com` / `chat.openai.com` are hard-routed to main `sdluffy` / `Sabbu` Chrome logged in as `sabarish.duvvuru@gmail.com`. Craft's in-app Chromium is structurally unreliable for ChatGPT because Cloudflare/security verification blocks it. Skip any active Photo project folder when the operator says it is in use.
