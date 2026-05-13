---
name: OpenAges IF Releases
description: Use when working with OpenAges IF, a minimalist local-first GTD desktop app for todo, schedule, notes, pomodoro, doc parsing, and multi-tab workflows, especially when checking releases or install options from github.com/openages/if_releases.
icon: icon.svg
metadata:
  category: productivity/apps
  family: github-derived
  lifecycle: active
  canonical_slug: openages-if-releases
  source_repo: openages/if_releases
  source_url: https://github.com/openages/if_releases
  source_resolved_from: Chrome recent GitHub history
---

# OpenAges IF Releases

## Source

- Repository: <https://github.com/openages/if_releases>
- Product site: <https://if.openages.com>
- Releases: <https://github.com/openages/if_releases/releases>
- Latest observed release: `v0.3.8` published 2025-03-31

## Purpose

IF is a minimalist, local-first GTD desktop app for todo, schedule, note, and pomodoro workflows.

Use this skill when the operator asks to evaluate, install, use, or track IF/OpenAges release information.

## Capabilities from README

- Todo workflows with list, kanban, square, mindmap, table, and flat views.
- Auto archive support.
- Simple Markdown editor called Note.
- Schedule workflows with week, timeline, month, and fixed views.
- Minimalist pomodoro timer for mindflow.
- Doc Parse miniapp for parsing documents to Markdown.
- Multi-tab system.
- macOS and Windows downloads.

## Latest release assets observed

For `v0.3.8`:

- macOS Apple Silicon: `IF-0.3.8-arm64.dmg`
- macOS Intel: `IF-0.3.8-x64.dmg`
- Windows x64: `IF-0.3.8-x64.exe`

## Common commands

Check latest release:

```bash
gh release list -R openages/if_releases --limit 5
gh release view -R openages/if_releases --json tagName,name,publishedAt,url,assets
```

Download latest macOS Apple Silicon DMG:

```bash
gh release download v0.3.8 -R openages/if_releases --pattern 'IF-0.3.8-arm64.dmg' --dir ~/Downloads
```

## Use guidance

- Treat IF as a candidate local-first productivity surface, not a code library.
- For installation tasks, choose Apple Silicon DMG on modern Apple laptops unless `uname -m` indicates `x86_64`.
- Do not assume sync/cloud behavior; README only states local-first.
- For detailed product behavior, fetch official docs from <https://if.openages.com/docs> before making claims.

## Verification

When updating this skill, verify against:

```bash
gh release list -R openages/if_releases --limit 5
```
