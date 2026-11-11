---
slug: "ui-components/foundation"
title: "Foundation UI"
description: "The owned MIOSA frontend primitives used by the platform application."
section: "Platform builders"
order: 9021
---

The MIOSA platform owns a Foundation UI primitive set exposed to the Svelte app through the `$foundation` alias.

Use this surface for platform screens before adding route-local buttons, inputs, cards, modals, tabs, tables, or page chrome.

## Owned primitive surface

| Primitive group | Components |
|---|---|
| Actions | `Button`, `PillButton` |
| Forms | `Input`, `RoundedInput`, `Textarea`, `Select`, `Checkbox`, `Toggle`, `RadioGroup`, `Slider` |
| Layout | `GlassCard`, `AppCard`, `Separator`, `ScrollArea`, `Breadcrumb` |
| Feedback | `Alert`, `Toast`, `Loading`, `Skeleton`, `Progress` |
| Overlays | `Modal`, `Popover`, `Tooltip`, `Menu` |
| Data display | `Table`, `Tabs`, `Accordion`, `Avatar` |

## Import path

```svelte
<script lang="ts">
  import { GlassCard, PillButton, Alert, Table } from "$foundation";
</script>
```

The alias is configured in the platform Svelte config and resolves to:

```text
src/lib/foundations-components/src/lib/ui
```

## Compute workspace audit

The Compute workspace was using this surface inconsistently. Shared Compute components often use `$foundation`, but several top-level Compute routes still hand-roll controls, tables, cards, and page shells with route-local CSS.

| Route | Current state |
|---|---|
| Projects | Mostly shared Compute components; low direct Foundation usage in the page. |
| Deploy | High custom UI; raw filters, buttons, selects, table, and route CSS remain. |
| Logs | Partial Foundation usage through `PillButton`; raw inputs and table remain. |
| Analytics | Visible cards and actions converted to `GlassCard` and `PillButton`. |
| Environment Variables | Partial Foundation usage through `PillButton`; raw modal, form, and table remain. |
| Domains | Partial Foundation usage through `PillButton`; raw modal and table remain. |
| Storage | High custom UI; raw tabs, search, buttons, card grid, and route CSS remain. |
| Databases | Partial Foundation usage through `PillButton`; raw cards and modal remain. |
| Sandboxes | Partial Foundation usage through `PillButton`; raw table and filter controls remain. |
| Computers | Partial Foundation usage through `PillButton`; raw table and filter controls remain. |

## Cleanup order

1. Convert route-level actions and empty/error states to `PillButton`, `Alert`, `Skeleton`, and shared empty-state components.
2. Replace raw form controls with `RoundedInput`, `Input`, `Select`, `Checkbox`, and `Toggle`.
3. Replace hand-rolled tabs and chips with `Tabs` or a shared Compute filter-chip component.
4. Replace raw tables and cards with Foundation table primitives or shared Compute table/card components.
5. Collapse repeated shell CSS into shared Compute layout primitives instead of copying page, console, and main CSS per route.

## Verification guardrail

Every migrated route should keep passing:

```bash
pnpm check
pnpm test:e2e tests/platform-shell-regression.spec.ts --project=chromium
pnpm build
```

## See also

- [UI Components overview](/docs/ui-components)
- [React components](/docs/ui-components/react)
- [Vue components](/docs/ui-components/vue)
- [Svelte components](/docs/ui-components/svelte)
