---
name: kashier-style-guide
description: Use when building, reviewing, or styling any UI for the Kashier payment product — web portal, native mobile app, or POS terminal — including pages, layouts, navigation, tables, pagination, lists, forms, or individual components.
---

# Kashier Design System

> Version 0.13 · Last updated 2026-07-28
> **Canonical full list page: Balances `lVNlJ8QUx3McAwnkJlXhHS` node `418:12514`** ("Payout accounts") — sidebar + topbar + CTA row + filter row + table + pagination measured in ONE composition. When any other source disagrees on page geometry, sidebar, topbar, filter row, table metrics, or pagination, **this frame wins** (specs in `figma-exact.md`).
> Other Figma sources: status chips & table cross-check (Terminals `D7Fru0LdD2bDmYyXwzu9nC`), entity list cards (Payment Links `Kw8SaHwCXqMlu6AbyRMeNq`, Customers `4LRW6jrxg8sgSIlgtJVN2S`), the POS system (`nhbdiaZtsdroBgJnRbhC77`), core tokens (`EVFzgKM9DhPqmhP9F0cGdw`), the mobile app (`D38J1PWGdciEbyJq5RrpCS`, `gWH3omAqpHlhjqkAGg9zx7`), and side drawers (Customer Portal — Circle-pay `8Xb0ZVxoTa5ctB77GGdR85`). The **HTML prototypes** are canonical only for what Figma doesn't define (filter-panel behavior) — they are NOT canonical for the sidebar or topbar, which `418:12514` now defines.
> Stack: CSS variables + Tailwind + Radix UI primitives (shadcn/ui architecture) · Heroicons · Outfit/Noto Sans (EN), Cairo/Noto Naskh Arabic (AR)

## Philosophy

**The design system is a vocabulary, not a cage.** Use the tokens and components as your building blocks, then compose screens with normal good UI judgment — the system tells you *what things look like*, you decide *how the screen is organized*. Pixel-exact reproduction is required only when the task names a specific Figma frame (then open `references/figma-exact.md`). Everywhere else, a well-composed responsive page that uses the tokens correctly beats a rigid copy of one desktop screenshot.

## Where to Look (open only what the task needs)

| Task | Open |
|------|------|
| Project setup, full palette, tailwind config, type scale | `references/tokens.md` |
| A specific component (Button, Input, Badge, Alert, Card, Avatar, Select, Checkbox, Switch, **ModeChip**, **Modal/Dialog**, **Drawer**, Tabs, Breadcrumb, AccountSelector, KpiBar), icons, RTL | `references/components.md` (Drawer lives in `references/patterns.md` §3) |
| **Any page or screen** — shell, nav, **topbar** (title + module search + language/currency + test/live mode), page header, templates, grids, breakpoints | `references/layout.md` |
| **A new feature, an overlay (drawer/modal), or "where should this live"** — disclosure ladder, feature-placement IA, drawer/modal/sheet specs, motion/focus/a11y, destructive-action rules | `references/patterns.md` |
| Tabular data — DataTable, sorting, **pagination** (compact bar, no page numbers), **status chips**, dashed filter chips, empty/loading states, responsive tables | `references/tables.md` |
| Lists that aren't tables — **entity record lists (RecordCard)**, selectable **PickerRow**, menus/settings/pickers, description lists | `references/lists.md` |
| Native mobile app or POS terminal, cross-platform decisions | `references/platforms.md` |
| Reproducing a named Figma frame pixel-for-pixel | `references/figma-exact.md` |

Building a full page means at least `layout.md` + whichever data/display files apply.

## Token Cheat Sheet (semantic — covers most work)

| Token | Value | Use for |
|-------|-------|---------|
| `primary` | #001F5F Ocean navy | Buttons, links, focus rings, active nav |
| `accent` | #00BCB4 Kashier teal | Accent buttons, highlights, progress |
| `secondary` | #FF9D6C Peachy | Secondary/warm highlights |
| `destructive` | #A50017 | Delete, errors, failed |
| `warning` | #FBAF1F | Pending, warnings |
| `success` | #4AAB4E | Paid, verified, completed |
| `foreground` | #202020 | Primary text |
| `muted-foreground` | #637373 | Hints, placeholders, secondary text |
| `cadet-base` | #556767 | Card titles, section/row labels |
| `border` | #E6E8E8 | **Inputs, table rows**, strong separators |
| `border-subtle` | #F5F6F6 | **Cards**, header dividers, light separators |
| `muted` | #F5F6F6 | Disabled bg, footers, table headers |
| `--link` / `ocean-80` | #2955B1 | Text links, linked table cells (ids, dates) |
| `--primary-hover` / `ocean-90` | #103788 | Primary button/link hover |
| `--primary-10` / `ocean-10` | #F1F6FF | Row hover, active page button, chips |
| radius | 4px (`rounded`) | Everything (alerts: 5px) |

Typography: `font-display` (Outfit/Cairo) for headings, `font-body` (Noto Sans/Noto Naskh Arabic) for everything else. Body UI text is `text-xs` (14px); page titles `text-h3` (39px) desktop / `text-h6`–`h5` mobile.

## Layout Principles (the 7 rules that make screens look right)

1. **Every screen has a skeleton**: `AppShell` (nav chrome) → content container → `PageHeader` → sections. A bare stack of components is a bug. Exceptions: auth pages and POS screens (own fullscreen layouts).
2. **Constrain content, in tiers — don't clamp everything to one width**: the canonical page geometry is `sidebar 293 + 40px gutter + content 1305 + 40px gutter` (Balances `418:12514`). List/dashboard pages are fluid up to `max-w-[1600px]` — that cap is reached right at a 1920 monitor, so 1920 fills edge-to-edge and only wider screens gain margin — detail pages to `max-w-[1305px]` (the designed width), forms to `max-w-2xl`, prose/settings to `max-w-3xl` — set via `AppShell`'s `size` prop (`layout.md` §2/§4). Padding scales `p-4 md:p-6 lg:p-8 xl:p-10`. Nothing stretches edge-to-edge unbounded, and nothing gets stuck at a phone-era 1200px on a wide monitor either.
3. **Spacing = relationship** (law of proximity): the gap between two things states how related they are — 4–8px inside a control · 8–12px tightly related controls · 16px members of a group/card · 24px between groups and page sections · 32–64px between regions. Related and unrelated neighbors must never share the same gap. Off-scale values round to the nearest step (4, 8, 12, 16, 20, 24, 32, 40, 48, 64).
4. **Designs adapt to screens** (Tailwind defaults, mobile-first): base = phone (drawer nav, lists, stacked full-width actions) → `md:` tables and 2-col grids return → `lg:` fixed sidebar and 2/3+1/3 detail layouts. A Figma frame shows ONE width — every fixed-width element in it (search fields, chips, stat cells) needs a defined smaller-screen fallback: shrink, collapse to an icon, wrap, or hide (e.g. the topbar's adaptive spec, layout.md §6b). A row that overflows or squashes at any width is a bug.
5. **Data has three states**: loaded, loading (skeleton), and empty (icon + message + CTA). Tables also need pagination. Shipping only the loaded state is incomplete work.
6. **Touch is not hover**: on mobile/POS every `hover:` has an `active:` press counterpart; touch targets ≥44px (POS: ≥48px).
7. **Disclosure before layout**: before picking a page template or a nav slot, run the disclosure ladder and the feature-placement table (`patterns.md` §1–2) — decide whether this is a nav item, a module tab, a settings section, or a drawer-only task, and whether it needs a page, a drawer, or a modal, BEFORE writing markup.

## UX Laws (how spacing & layout decisions get made)

Apply these when composing anything the reference files don't prescribe exactly:

1. **Proximity** — distance encodes relationship, horizontally AND vertically (see Layout Principle 3's gap scale). Horizontal: topbar groups — 12px inside the locale pair, 16px between utility icons, 24–32px between groups. Vertical: an element sits closest to what it owns — filter strip 12px above its table, section header 8px above its content, KPI label 0px above its value — and 32px from unrelated regions (topbar↔page, section↔section). One uniform gap down a page is a proximity violation.
2. **Common region** — a border/card groups its contents; one card = one concern. Don't mix unrelated content inside a card (filters live OUTSIDE the table card for this reason).
3. **Similarity** — same role, same look: every filter is a `FilterChip`, every status a `StatusChip`, every entity list a `RecordCard`. A one-off styling for a repeated role is a bug.
4. **Visual hierarchy** — exactly ONE primary action per view; rank importance with size/weight/color (title 39 bold > section 16 bold > label 12 gray; amounts bold, metadata muted). If everything is emphasized, nothing is.
5. **Fitts's law** — frequent actions get large, close targets (40px topbar create, 96px POS buttons, ≥44px touch); destructive actions sit away from primary paths (bottom of sheets, in red).
6. **Hick's law** — limit simultaneous choices: ≤5 nav groups, ≤5 visible filter chips, overflow behind "More"/kebab/sheets; progressive disclosure for advanced options (filter panel, settings menu). This is also why not every feature earns a nav slot — run the placement table (`patterns.md` §2) before adding one; a 6th nav group is a Hick's-law violation even if the feature is important.
7. **Jakob's law** — reuse this skill's components and platform conventions before inventing anything new; this includes reaching for the `Drawer` (`patterns.md` §3) before a bespoke overlay, since drawers-for-tasks are now the established pattern across web and mobile.
8. **Miller's law** — chunk information: day-grouped lists, ~4-item meta rows, sectioned cards — never a wall of undifferentiated data.

## Component Rules

1. **Token-first** — every color, size, spacing, radius maps to a CSS variable or Tailwind token. No raw hex in component code (primitives like `ocean-10` are fine; hex literals only where `tokens.md` has no name for the value).
2. **shadcn structure** — `cva()` for variants; export `ComponentProps` and `componentVariants`.
3. **Radix primitives** for interactive components (Dialog, Select, Checkbox, Switch, Tabs, DropdownMenu, Tooltip) — never hand-rolled.
4. **Heroicons only, verbatim** — import `@heroicons/react`, or copy the exact SVG from heroicons.com; NEVER hand-write or approximate path data (that's how broken icons happen). 18px default (nav, buttons; nav at stroke 1.5), 16px small, 24px topbar/alerts/headers.
5. **Focus rings** — `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`.
6. **RTL-safe** — logical properties only (`ms-`/`me-`, `ps-`/`pe-`, `text-start`/`text-end`, `rtl:rotate-180` on directional chevrons). Hardcoded `left`/`right` is a bug. Arabic sizes: xSmall 13px, xxSmall 10px.
7. **States required** — default, hover (pointer), active (touch), focus-visible, disabled; error for inputs; selected for tabs/nav/list items.
8. **Semantic elements** — clickable things are `<button>`/`<a>`, never `<div onClick>`.

## Before You Ship a Screen (mandatory output check)

Walk EVERY delivered screen through this list at 360, 768, 1024, 1440, **and 1920px** — not just
the width you built at. A defect at any width is a defect in the output:

1. **No horizontal overflow** at any width; **nothing overlaps, clips, or collides** — check
   text against its container (long amounts, long names) and fixed elements against each other.
2. **No duplicated chrome** — one header, one search, one nav per viewport; components that
   collapse on mobile actually collapse in code (`hidden md:flex`), not just in intent.
3. **Type steps down** where containers shrink (39→31→20 titles, 31→25 stat values).
4. **Proximity gradient is visible** — within-group < between-group gaps, owner+owned pairs tight.
5. **All states exist** — loading, empty, error; hover AND press feedback.
6. **Icons are verbatim Heroicons**; RTL logical properties throughout.
7. **Wide screens checked, not just capped** — at 1920px the content container fills to its tier's
   cap (`max-w-[1600px]` list/dashboard, `max-w-[1305px]` detail) instead of sitting stuck at
   1200px with dead margins, AND instead of stretching fully unbounded (layout.md §2).
8. **Overlays are complete, not just present** — every drawer/modal scroll-locks the page behind
   it, has a sticky footer if it has actions, returns focus to its trigger on close, and collapses
   to a bottom sheet `<md` (patterns.md §3–4). A drawer was used for record detail/tasks/lists; a
   modal only for the three cases the ladder reserves for it (patterns.md §1).
9. **Canonical metrics match `418:12514`** — sidebar 293, nav item 44, topbar 47, search a 370px
   pill, filter chip 34, table header 40, row 49, pagination 44 (figma-exact.md). Eyeballed
   substitutes for these are a defect.

When the user corrects an output, the mistake gets recorded here or in Anti-Patterns —
the same defect must never ship twice.

## Anti-Patterns (the failures this skill exists to prevent)

| ❌ Failure | ✅ Instead |
|-----------|-----------|
| Numbered page buttons in pagination | Compact bar: count · rows-per-page · prev/next + "1/10" fraction (tables.md §3) |
| Bold pill badges for table/POS status | `StatusChip` — radius 2, Noto Sans regular uppercase (tables.md) |
| Entity list (customers, payment links…) built as a table or divider rows | `RecordCard` groups — bordered cards with day headers (lists.md §1) |
| Hand-drawn or "approximate" icon SVG paths | Verbatim Heroicons: `@heroicons/react` import or exact copy from heroicons.com |
| Two searches on one page (topbar + table toolbar) | The topbar module search is THE page search (layout.md §6b) |
| Per-cell borders on stat/KPI cells (doubled 2px lines) | Container border + `gap-px` single dividers (components.md KpiBar) |
| Desktop topbar controls rendered on mobile (double header, overlap) | Topbar collapses to title-only `<md` IN CODE; controls move to drawer (layout.md §6b) |
| Fixed desktop type sizes overflowing mobile cells (31px stat values) | Responsive step-down: `text-h5 md:text-h4` + `min-w-0` + break-words |
| Rounded-pill filter buttons; same border for set/unset filters | `FilterChip` radius-4: dashed border = empty, SOLID border + value = applied (tables.md) |
| Components stacked with no shell or page header | `AppShell` + `PageHeader` (layout.md §1) |
| Cards/tables spanning a 1920px window unbounded | Tiered container per template, `size` prop (layout.md §2) |
| `max-w-[1200px]` on every page, leaving 360px dead margins on a 1920 monitor | Fluid tier: `max-w-[1600px]` list/dashboard, `max-w-[1305px]` detail (layout.md §2) |
| **240px sidebar / 56px table rows / 12px filter-chip labels** — invented metrics | Canonical `418:12514`: sidebar **293**, nav item **44** (48 pitch), row **49**, header **40**, filter chip **34** with a **14px** label (figma-exact.md) |
| **A title + Export toolbar bar inside the table card** on a list page | Canonical list page has NO toolbar: CTA is its own end-aligned row, **Export sits at the end of the FILTER row** outside the card (tables.md §1) |
| Single-line table cells for identity columns | `TwoLineCell` — primary 14px over muted 12px secondary (`Misr Gadida 1 branch` / `ACCXXXX`), the canonical default (tables.md) |
| Dropping "Developers" from the topbar; rendering the cog as an outline icon | Developers IS in the canonical topbar; **cog-6-tooth is the SOLID variant**, bell + question-mark stay outline (layout.md §6b) |
| Square/radius-4 topbar search field | Search is a **370×40 pill** (`rounded-full`), 24px icon 16px from the end (layout.md §6b) |
| Tiny 11px uppercase gray sidebar section labels | "Manage" is **SemiBold ~15px foreground**, indented 16px (layout.md §5) |
| Modal used for record detail, a multi-field/multi-step task, or browsing/selecting from a list | `Drawer` — `DetailDrawer`/`TaskDrawer` (patterns.md §1, §3) |
| New feature dropped into the sidebar without checking if it belongs there | Run the feature-placement table first (patterns.md §2) |
| 620px/480px desktop drawer rendered full-size on a 360px phone | Collapses to a bottom sheet `<md` (patterns.md §3c) |
| Desktop sidebar squeezed onto a phone | Drawer nav `<lg` (layout.md §4) |
| Table with no pagination, cut off at N rows | `DataTable` + `TablePagination` (tables.md) |
| Blank white box while data loads / when empty | Skeleton + `EmptyState` |
| Full `DataTable` rendered on mobile | Horizontal scroll or list collapse (tables.md §5) |
| `hover:` styles as the only feedback on touch | `active:` press states |
| Fixed pixel widths on content (`w-[850px]`) | Fractions, `flex-1`, `max-w-*`, `min-w-0` |
| `ml-4`, `text-left`, `pl-3` | `ms-4`, `text-start`, `ps-3` (RTL) |
| Raw hex colors sprinkled in JSX | Semantic tokens / primitive palette classes |

## File Structure Convention

```
components/
  ui/           ← primitives (Button, Input, Badge, Card, DataTable, ListView, …)
  features/     ← Kashier compositions (TransactionTable, PaymentForm, …)
  layouts/      ← AppShell, Sidebar, PageHeader
lib/utils.ts    ← cn() helper
styles/globals.css ← :root variables (references/tokens.md)
```
