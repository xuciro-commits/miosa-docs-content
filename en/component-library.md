---
slug: "component-library"
title: "Foundation Component Library"
description: "MIOSA Foundation UI reference with design intent, Tailwind v4 tokens, CSS variables, and design tokens."
section: "Platform builders"
order: 9020
---

MIOSA Foundation is the internal component library for the platform frontend. It is the place to inspect components before deciding what to keep, remove, rename, or rebuild.

This is the product UI system for building MIOSA itself, not the drop-in `@miosa/*` packages that customers import to embed sandboxes in their own apps  -  those are documented in [UI Components](/docs/ui-components).

The live browser is maintained in the Foundation package:

```bash
cd miosa-platform/miosa-frontend/src/lib/foundations-components
npm run dev -- --host 127.0.0.1 --port 5174
```

Open:

```text
http://127.0.0.1:5174/component-library
```

> **Note: Current intent**
>
> The library is being cleaned up from a broad prototype set into a small, repeatable MIOSA UI system. Treat this page as the design contract for that cleanup work.

## What Lives Where

- [Live component browser](http://127.0.0.1:5174/component-library): Browse every section, pattern, and demo in the Foundation package.
- **Source sections**: `src/lib/foundations-components/src/routes/component-library/sections`
- **Design reference**: `src/lib/foundations-components/DESIGN.md`
- **Cleanup review**: `src/lib/foundations-components/COMPONENT_REVIEW.md`

## Compact Reference

MIOSA UI should feel like an operator console for AI compute: precise, dark, dense, and fast to scan. Keep the system mostly neutral. Use a limited blue/cyan signal color only for active compute, selected state, focus, and live status. Avoid decorative gradients, oversized marketing sections, duplicate button variants, and one-off component styling.

| Token | Value | Role |
|---|---:|---|
| Canvas | `#07090d` | App background |
| Panel | `#10141c` | Primary component surface |
| Raised | `#151b25` | Elevated controls and selected rows |
| Border | `#263041` | Hairline structure |
| Text | `#f4f7fb` | Primary text |
| Muted | `#8d98aa` | Secondary labels |
| Signal | `#60a5fa` | Active compute, focus, selected state |
| Success | `#34d399` | Healthy/running status |
| Warning | `#fbbf24` | Attention state |
| Danger | `#f87171` | Destructive or failed state |

Core rules:

- Use one primary button per visual region.
- Prefer compact toolbars, segmented controls, tabs, tables, and split panes.
- Use 1px borders for structure; use shadows sparingly.
- Use status color only where state must be read quickly.
- Keep cards at `8px` radius unless the existing local pattern requires less.
- Remove duplicate button/component variants unless they encode meaning.

## Extended Reference

**DESIGN.md**

```md
# MIOSA Foundation UI

Theme: dark operator console

MIOSA Foundation is a component system for AI compute, white-label workspaces,
sandboxes, deployments, file browsers, agents, and usage operations. It should
feel like a precise product surface, not a marketing page.

## Design Character

The UI is compact, structured, and status-rich. Components should look like
they belong in the same control plane: split panes, command bars, tables,
logs, terminals, inspectors, file trees, detail drawers, and focused panels.

The component library should not keep every prototype. It should converge on
the smallest set of primitives and domain components that can compose the real
MIOSA product.

## Keep

- Buttons with clear hierarchy: primary, secondary, ghost, icon, destructive.
- Inputs, selects, textareas, checkboxes, switches, sliders, tabs, menus.
- Tables and data grids for operational records.
- Split panes for preview + files, terminal + logs, list + detail.
- Status badges for sandbox, deployment, compute, billing, and security states.
- Drawers and dialogs for contained workflows.
- Command palette and Ask OSA controls.
- File tree, terminal, preview iframe, usage summary, and deployment panels.

## Remove Or Merge

- Duplicate button families that only differ visually.
- Decorative card variants with no product role.
- Repeated chat/input shells that should become one canonical agent composer.
- Demo-only modules that do not map to MIOSA product workflows.
- Any component that requires a special color, radius, or font rule by itself.

## Component Quality Bar

Each kept component needs:

- A real product role.
- A stable API.
- Keyboard/focus behavior where relevant.
- Loading, empty, error, disabled, and selected states.
- A single source of styling truth.
- No hidden dependency on app-only stores unless wrapped behind local adapters.
```

**Tailwind v4**

```css
@theme {
  /* Colors */
  --color-miosa-canvas: #07090d;
  --color-miosa-panel: #10141c;
  --color-miosa-raised: #151b25;
  --color-miosa-border: #263041;
  --color-miosa-border-soft: #1c2533;
  --color-miosa-text: #f4f7fb;
  --color-miosa-muted: #8d98aa;
  --color-miosa-dim: #5f6b7d;
  --color-miosa-signal: #60a5fa;
  --color-miosa-signal-soft: #1d4ed8;
  --color-miosa-success: #34d399;
  --color-miosa-warning: #fbbf24;
  --color-miosa-danger: #f87171;

  /* Typography */
  --font-miosa-sans: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-miosa-mono: "SF Mono", "Cascadia Code", ui-monospace, monospace;

  --text-miosa-caption: 12px;
  --text-miosa-body: 14px;
  --text-miosa-ui: 13px;
  --text-miosa-title: 18px;
  --text-miosa-display: 40px;

  /* Shape */
  --radius-miosa-xs: 4px;
  --radius-miosa-sm: 6px;
  --radius-miosa-md: 8px;
  --radius-miosa-pill: 9999px;

  /* Spacing */
  --spacing-miosa-1: 4px;
  --spacing-miosa-2: 8px;
  --spacing-miosa-3: 12px;
  --spacing-miosa-4: 16px;
  --spacing-miosa-6: 24px;
  --spacing-miosa-8: 32px;
}
```

**CSS Variables**

```css
:root {
  color-scheme: dark;

  --miosa-canvas: #07090d;
  --miosa-panel: #10141c;
  --miosa-raised: #151b25;
  --miosa-border: #263041;
  --miosa-border-soft: #1c2533;
  --miosa-text: #f4f7fb;
  --miosa-muted: #8d98aa;
  --miosa-dim: #5f6b7d;
  --miosa-signal: #60a5fa;
  --miosa-signal-soft: #1d4ed8;
  --miosa-success: #34d399;
  --miosa-warning: #fbbf24;
  --miosa-danger: #f87171;

  --miosa-font-sans: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --miosa-font-mono: "SF Mono", "Cascadia Code", ui-monospace, monospace;

  --miosa-text-caption: 12px;
  --miosa-text-ui: 13px;
  --miosa-text-body: 14px;
  --miosa-text-title: 18px;
  --miosa-text-display: 40px;

  --miosa-leading-tight: 1.2;
  --miosa-leading-ui: 1.45;
  --miosa-leading-body: 1.55;

  --miosa-radius-xs: 4px;
  --miosa-radius-sm: 6px;
  --miosa-radius-md: 8px;
  --miosa-radius-pill: 9999px;

  --miosa-space-1: 4px;
  --miosa-space-2: 8px;
  --miosa-space-3: 12px;
  --miosa-space-4: 16px;
  --miosa-space-6: 24px;
  --miosa-space-8: 32px;
}
```

**Design Tokens**

```json
{
  "color": {
    "canvas": { "$value": "#07090d", "$type": "color" },
    "panel": { "$value": "#10141c", "$type": "color" },
    "raised": { "$value": "#151b25", "$type": "color" },
    "border": { "$value": "#263041", "$type": "color" },
    "border-soft": { "$value": "#1c2533", "$type": "color" },
    "text": { "$value": "#f4f7fb", "$type": "color" },
    "muted": { "$value": "#8d98aa", "$type": "color" },
    "dim": { "$value": "#5f6b7d", "$type": "color" },
    "signal": { "$value": "#60a5fa", "$type": "color" },
    "signal-soft": { "$value": "#1d4ed8", "$type": "color" },
    "success": { "$value": "#34d399", "$type": "color" },
    "warning": { "$value": "#fbbf24", "$type": "color" },
    "danger": { "$value": "#f87171", "$type": "color" }
  },
  "font": {
    "sans": { "$value": "Inter", "$type": "fontFamily" },
    "mono": { "$value": "SF Mono", "$type": "fontFamily" }
  },
  "typography": {
    "caption": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "12px", "fontWeight": 500, "lineHeight": 1.45 }
    },
    "ui": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "13px", "fontWeight": 500, "lineHeight": 1.45 }
    },
    "body": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "14px", "fontWeight": 400, "lineHeight": 1.55 }
    },
    "title": {
      "$type": "typography",
      "$value": { "fontFamily": "Inter", "fontSize": "18px", "fontWeight": 600, "lineHeight": 1.2 }
    }
  },
  "radius": {
    "xs": { "$value": "4px", "$type": "dimension" },
    "sm": { "$value": "6px", "$type": "dimension" },
    "md": { "$value": "8px", "$type": "dimension" },
    "pill": { "$value": "9999px", "$type": "dimension" }
  },
  "spacing": {
    "1": { "$value": "4px", "$type": "dimension" },
    "2": { "$value": "8px", "$type": "dimension" },
    "3": { "$value": "12px", "$type": "dimension" },
    "4": { "$value": "16px", "$type": "dimension" },
    "6": { "$value": "24px", "$type": "dimension" },
    "8": { "$value": "32px", "$type": "dimension" }
  }
}
```

## Component Cleanup Checklist

Use this checklist when deciding what stays in the Foundation library.

| Decision | Keep When | Remove Or Merge When |
|---|---|---|
| Button variant | It maps to a real action hierarchy | It only changes color, radius, or vibe |
| Card/panel | It represents a reusable product surface | It is a one-off demo wrapper |
| Input/composer | It handles real data entry or agent prompts | It duplicates another composer shell |
| Table/list | It supports scan, sort, compare, or operate workflows | It is only visual filler |
| Status badge | It encodes real state | It is decorative labeling |
| Module page | It maps to a MIOSA product workflow | It is a generic SaaS mockup |

## See Also

- [UI Components overview](/docs/ui-components)
- [React UI Components](/docs/ui-components/react)
- [Svelte UI Components](/docs/ui-components/svelte)
- [Vue UI Components](/docs/ui-components/vue)
