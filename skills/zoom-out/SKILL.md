---
name: Zoom Out
description: 'Use when the user needs: Tell the agent to zoom out and give broader context or a higher-level perspective. Use when you''re unfamiliar with a section of code or need to understand how it fits into the bigger picture.'
icon: icon.svg
disable-model-invocation: true
metadata:
  category: memory/context
  family: memory
  lifecycle: active
  canonical_slug: zoom-out
  icon_style: craft-category-glyph-v1
---

I don't know this area of code well. Go up a layer of abstraction. Give me a map of all the relevant modules and callers.

## Pipeline

```
Input (current file/path/module or "zoom out" trigger)
  ↓
Identify Current Scope (file → directory → package → system)
  ↓
Map Upward Layers:
  ├── Parent module / package
  ├── Callers and importers
  ├── Dependents (who depends on this)
  ├── Sibling modules (same level)
  └── System-level role (where does this fit)
  ↓
Synthesis (architecture map, dependency graph, role summary)
  ↓
Artifact Generation
  ├── Inline map (default, compact)
  └── ShadowArchive/80-reports/code-map-<module>-YYYY-MM-DD.md  (when requested)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Module map: parent, siblings, callers, dependents, system role | `zoom out` trigger or unfamiliarity signal |
| `brief` | One-paragraph summary of where this module fits | Quick orientation |
| `full` | Full dependency graph with Mermaid diagram + all callers + all dependents | Deep understanding needed |
| `callers` | Only who calls this module (no other context) | Narrow question |
| `siblings` | Only modules at the same level | Exploring a layer |
| `system` | Only the system-level role and architecture fit | High-level positioning |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Code map (full) | `ShadowArchive/80-reports/code-map-<module>-YYYY-MM-DD.md` | Durable architecture reference |
| Dependency graph | Inline Mermaid or ASCII (not saved unless requested) | Visual map |

## Fallback Chain

1. **Primary:** AST analysis (grep imports, find callers, read package.json/cargo.toml/pyproject.toml) → map layers → synthesize
2. **No language tooling available:** Fall back to `grep` for imports/requires/uses + `find` for file structure
3. **Monorepo with many packages:** Scope to the relevant package first, then map upward; do not map the entire monorepo
4. **Binary/compiled module:** Report that source is not available; map from type definitions, header files, or API surface if possible
5. **Last resort:** File system structure only (`find` + `ls`); explicitly state that no semantic analysis was possible

## Prerequisites

- Access to the codebase (local filesystem or git repo)
- `grep`/`ripgrep` for import/caller analysis
- Language-appropriate package manifest (package.json, Cargo.toml, pyproject.toml, go.mod) for dependency mapping
- Mermaid rendering support (for full mode diagrams)

## Error Handling

| Failure | Recovery |
|---------|----------|
| File not found | Report; suggest possible paths; do not invent structure |
| No imports/exports detected | Map file structure only; note that the module may be entry-point or standalone |
| Circular dependency detected | Flag in output; do not attempt to flatten; show the cycle explicitly |
| Too many callers (>50) | Show top 10 + count; offer full list on request |
| Monorepo too large to map | Scope to nearest package boundary; note scoped output |

## Contract

- **Read-only.** zoom-out never modifies code, creates files (except reports), or runs build/test commands.
- **Orientation, not refactoring.** The purpose is understanding. Do not propose or execute code changes from zoom-out.
- **Honest about gaps.** If the module is unfamiliar, say so. Do not fabricate architecture knowledge.
- **Externalization rule.** Full code maps go to `ShadowArchive/80-reports/` only when requested. Default output is inline only.
- **Do not** execute zoom-out recursively beyond 3 levels upward (file → package → system). Deeper exploration requires explicit operator request.
- **Do not** analyze test files for caller maps unless the operator asks. Focus on production code paths.

## Pipeline

```
Current focus → Zoom out one level → Identify adjacent surfaces → Report
```

1. Identify current work focus (file, skill, program, context zone)
2. Zoom out one level (project → program → org → fleet)
3. Identify adjacent surfaces that should be aware of current work
4. Report gaps and next actions

## Modes

| Mode | Zoom level | When |
|------|-----------|------|
| `one` (default) | One level up | "zoom out", "what else" |
| `full` | Full context map | "full picture" |

## Contract

- Zoom out, not explode — one level, not full repo scan
- Report gaps as questions, not mandates
