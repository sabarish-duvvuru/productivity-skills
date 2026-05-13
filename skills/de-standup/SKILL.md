---
name: 'DE Standup'
description: 'Use when the user needs: US DE Weekly Stand-up system — Monday dashboard, Thursday PPTX report, engineer update form. Run refresh, status, export, or form commands. Trigger on "standup", "weekly report", "de standup", "refresh dashboard", "export report", "weekly status".'
icon: icon.svg
metadata:
  category: enterprise/work
  family: enterprise
  lifecycle: active
  canonical_slug: de-standup
  icon_style: craft-category-glyph-v1
---

# de-standup — US DE Weekly Stand-up & Report System

Meeting operating system for the US DE Inverter/IPM team.

**Team structure and leadership are loaded dynamically from shadow-sidecast:**

```bash
# Get US DE team structure and leadership
shadow-sidecast team-map --program FH4S
shadow-sidecast team-map --program FH4
shadow-sidecast broadcast-org --scope usgrp1
```

**This returns:**
- US DE lead and team structure (Inverter & IPM teams)
- Team members with roles and program assignments
- Cross-functional stakeholders and reporting lines

## Commands

| Input | Action |
|-------|--------|
| No args / "refresh" | Seed programs + generate HTML dashboard |
| "status" | Print RAG status per program + open actions |
| "export" | Generate PPTX report with Astemo branding |
| "form" | Start HTTP update form on localhost:8090 |

## Usage

```bash
# Refresh dashboard (runs daily at 9 AM via cron)
PYTHONPATH=tools python3.14 -m de_standup.cli refresh

# Check current week status
PYTHONPATH=tools python3.14 -m de_standup.cli status

# Generate Thursday PPTX report
PYTHONPATH=tools python3.14 -m de_standup.cli export

# Start update form for engineers (Thu-Fri)
PYTHONPATH=tools python3.14 -m de_standup.cli form
# → http://localhost:8090/?program=FH4S&pic=Sabarish

# Override week or database
PYTHONPATH=tools python3.14 -m de_standup.cli --week 2026-W14 --db /tmp/test.db status
```

## Weekly Cycle

```
Thursday noon  → Report auto-generated, Kevin reviews
Thursday noon  → Update form opens for next week
Friday evening → Update form closes
Daily 9:00 AM  → Dashboard snapshot refreshed (launchd cron)
Monday 9:00 AM → DE Stand-up meeting uses dashboard
```

Cancelled-closeout programs do not participate in the normal weekly refresh, update-form, or report cadence unless the operator explicitly wants close-out reporting.

## Programs (from xEV Classification)

**Program PIC assignments and team structure are loaded dynamically from shadow-sidecast:**

```bash
# Get program mappings and PIC assignments
shadow-sidecast team-map --program FH4S
shadow-sidecast team-map --program FH4
shadow-sidecast team-map --program FH4L
shadow-sidecast team-map --program VEC
```

**This returns:**
- Program PICs (Person In Charge) with roles and program assignments
- Team members and their program associations
- Cross-functional stakeholders and program status
- Reporting lines and team structure

Current program classifications (from xEV):
| Program | Status |
|---------|--------|
| FH4 | Production |
| FH4S | Active dev |
| FH4L | Early |
| XP7G | Cancelled by Honda; close-out only |
| XY9H | Pre-dev |
| VEC | Ongoing |

For current PIC assignments and team structure, reference shadow-sidecast above.

## Data Sources

- SharePoint Lists: US DE tasks (13), VEC (7), QA-Sheet (23)
- SharePoint Files: 1,185 on USGRPxEV, 8,957 on NAGEN4VAFH4S
- Local: Change Point Tracker, Smart Drawings, Kernel
- Smartsheet: DE Activity Schedule (ID: 2231837406482308)

## Output

- **Dashboard**: `tools/de_standup/reports/standup-{week}.html`
- **PPTX**: `tools/de_standup/reports/standup-{week}.pptx`
- **DB**: `tools/de_standup/standup.db`

## Architecture

21 tests, 8 modules: models.py, seed.py, sharepoint.py, dashboard.py, export.py, cli.py, form.py + templates.
