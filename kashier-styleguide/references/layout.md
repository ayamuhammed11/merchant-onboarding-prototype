# Kashier Layout System — Shell, Page Templates, Grids

> Load this file whenever building a page or screen (not just a single component).
> Platform-specific shells (mobile app, POS) are in `platforms.md`.

## 1. Page Anatomy — every screen has these layers

```
AppShell (nav chrome: sidebar ≥lg, top bar + drawer <lg)
 └─ <main>  bg-white, scrollable
     └─ Content container  (max-width + responsive padding) — vertical gaps encode ownership:
         ├─ Topbar                     ]
         │   ↓ 32px                    ] the topbar governs the whole page → region gap
         ├─ Section (KPIs / cards)     ]
         │   ↓ 32px                    ] independent sections → large gap
         ├─ Filter strip               ]
         │   ↓ 12px                    ] the filters OWN the table below → tight gap
         ├─ Table / list               ]
         │   ↓ 32px
         └─ Next section…
```

The page stack is `gap-8` (32px) between top-level regions; anything that OWNS what follows
(filter strip → table, section header → its content) is wrapped WITH it in a tighter
sub-stack (`gap-2`/`gap-3`) — never rely on one uniform gap for the whole page.

Never render a bare stack of components — a page without `AppShell` + a topbar/`PageHeader` is a bug unless it's a public/auth page (login, signup, onboarding) or a POS screen.

## 2. Spacing Rhythm & Content Constraint

Spacing is the law of proximity made concrete — **each step means a relationship**, and related vs unrelated neighbors must never share the same gap:

| Gap | Relationship it states | Examples |
|-----|------------------------|----------|
| 4–8px (`gap-1`/`gap-2`) | Parts of ONE control | chip icon↔label, currency↔chevron, title↔subtitle |
| 8–12px (`gap-2`/`gap-3`) | Tightly related controls; an element and what it OWNS | footer buttons, locale group (عربي↔EGP), filter strip↔its table, section header↔its content |
| 16px (`gap-4`) | Members of a group / inside a card | utility icon cluster, card padding, filter chips row |
| 24px (`gap-6`) | Between groups | topbar groups, cards in a grid |
| 32–64px (`gap-8`–`gap-16`) | Between regions & page sections | topbar↔page body, section↔section, topbar start↔controls, sidebar↔content |

**Content must be constrained, not clamped.** A single hardcoded `max-w-[1200px]` on every page is
its own failure at wide sizes — it leaves 360px of dead margin on each side of a 1920px monitor
(700px on a 2560px monitor). The fix is a **tier per template**, not a single fixed number:

**The canonical desktop frame** (Balances `418:12514`, 1678px wide) measures:
`sidebar 293 + 40px gutter + content 1305 + 40px gutter = 1678`. So the desktop gutter is **40px**
and the designed content width is **1305px** — the tiers below keep that geometry and let the
content grow from it rather than inventing a new one.

| Template | Container | Rationale |
|----------|-----------|-----------|
| List / dashboard | `w-full max-w-[1600px] mx-auto` | Fluid. The 1600 cap is reached right AT a 1920 monitor (`1920 − 293 sidebar ≈ 1612`), so 1920 is filled edge-to-edge and anything wider turns into centered margin — exactly the "fluid to 1920, constrained beyond" behavior |
| Detail (record pages) | `w-full max-w-[1305px] mx-auto` | The canonical Figma content width — 2/3+1/3 detail grids were designed at exactly this |
| Form | `max-w-2xl` (672px) | Fixed — form line length shouldn't grow with the monitor |
| Prose / settings | `max-w-3xl` (768px) | Fixed — same reasoning as forms |

| Rule | Value |
|------|-------|
| Content area padding | `p-4` mobile → `md:p-6` → `lg:p-8` → `xl:p-10` (40px, canonical) |

Always wrap page content in the tiered container — never leave it unbounded, and never default
every page to the same width:

```tsx
<main className="flex-1 min-w-0 overflow-y-auto">
  {/* size="wide" (list/dashboard, default) | "detail" | "form" | "prose" — see AppShell §4 */}
  <div className="mx-auto w-full max-w-[1600px] p-4 md:p-6 lg:p-8 xl:p-10 flex flex-col gap-6">
    {/* PageHeader + sections */}
  </div>
</main>
```

**Overlays (drawers/modals) are a separate concern** from page-content width — they're sized by
their own component spec, not this table. See `patterns.md` §3.

## 3. Breakpoints (Tailwind defaults)

| Breakpoint | Width | Layout behavior |
|-----------|-------|-----------------|
| (base) | <640 | Single column. Top bar + nav drawer. Tables → lists/cards. Actions full-width. Side drawers become bottom sheets (`patterns.md` §3c). |
| `sm` | ≥640 | Still single column; inline action buttons allowed. |
| `md` | ≥768 | Full tables return. 2-col grids. Side drawers switch from bottom sheet to end-anchored panel. |
| `lg` | ≥1024 | Fixed sidebar appears. Detail pages go 2/3 + 1/3. |
| `xl` | ≥1280 | Topbar reaches full spec (`layout.md` §6b). Container padding steps up to `xl:p-10` (40px, the canonical gutter). |
| `2xl` | ≥1536 | Nothing new — the container is still fluid here. |
| `3xl` | ≥1920 | The list/dashboard container reaches its `max-w-[1600px]` cap right at 1920 (`1920 − 293 sidebar ≈ 1612`), so a 1920 monitor is filled edge-to-edge. Beyond this, extra width becomes centered margin. |

Design **mobile-first**: write base classes for the phone layout and add `md:` / `lg:` upgrades.
`3xl` requires adding `screens: { "3xl": "1920px" }` to the Tailwind config — see `tokens.md` §2.

## 4. AppShell (responsive)

Desktop ≥`lg`: fixed 240px sidebar. Below `lg`: a top bar with the **account switcher** at the start (38px logo, radius 4 + Noto Sans Bold 14px name + 20px chevron-down) and, at the end, a 24px bell icon + a 32px `Bars3BottomRightIcon` menu button that opens the sidebar as a slide-in drawer (Radix Dialog). (Source: Customers mobile Figma `455:9921`.)

```tsx
// components/layouts/AppShell.tsx
"use client";
import { useState } from "react";
import * as Dialog from "@radix-ui/react-dialog";
import { Bars3BottomRightIcon, BellIcon, ChevronDownIcon, XMarkIcon } from "@heroicons/react/24/outline";
import { Sidebar } from "./Sidebar";
import { cn } from "@/lib/utils";

// Container tiers — layout.md §2. "wide" is the default for list/dashboard pages.
const CONTENT_SIZES = {
  wide:   "max-w-[1600px]",   // list/dashboard — fluid, still fills 1920, caps past ~1975
  detail: "max-w-[1305px]",   // record detail — the canonical Figma content width
  form:   "max-w-2xl",        // 672px — fixed, line-length matters more than width
  prose:  "max-w-3xl",        // 768px — settings, prose
};

export function AppShell({
  children,
  sidebarProps,
  accountName = "Misr Elkheir",
  initials = "ME",
  size = "wide",
}: {
  children: React.ReactNode;
  sidebarProps?: React.ComponentProps<typeof Sidebar>;
  accountName?: string;         // mobile top bar account switcher
  initials?: string;
  size?: keyof typeof CONTENT_SIZES;   // picks the content container tier (layout.md §2)
}) {
  const [open, setOpen] = useState(false);
  return (
    <div className="flex min-h-screen bg-background">
      {/* Desktop sidebar */}
      <div className="hidden lg:block">
        <Sidebar {...sidebarProps} />
      </div>

      <div className="flex-1 flex flex-col min-w-0">
        {/* Mobile / tablet top bar: account switcher · bell · menu */}
        <header className="lg:hidden sticky top-0 z-40 h-14 flex items-center gap-2 px-4 bg-white border-b border-border-subtle">
          <button className="flex items-center gap-2 py-1.5 rounded-lg min-w-0 flex-1 text-start">
            <div className="w-[38px] h-[38px] rounded bg-accent flex items-center justify-center text-white text-xs font-bold shrink-0">
              {initials}
            </div>
            <span className="font-body text-sm font-bold text-foreground truncate">{accountName}</span>
            <ChevronDownIcon className="w-5 h-5 shrink-0 text-foreground" />
          </button>
          <button aria-label="Notifications" className="w-11 h-11 flex items-center justify-center rounded text-foreground hover:bg-muted">
            <BellIcon className="w-6 h-6" />
          </button>
          <Dialog.Root open={open} onOpenChange={setOpen}>
            <Dialog.Trigger asChild>
              <button
                aria-label="Open navigation"
                className="w-11 h-11 -me-2 flex items-center justify-center rounded text-foreground hover:bg-muted"
              >
                <Bars3BottomRightIcon className="w-8 h-8" />
              </button>
            </Dialog.Trigger>
            <Dialog.Portal>
              <Dialog.Overlay className="fixed inset-0 z-40 bg-black/40" />
              <Dialog.Content className="fixed inset-y-0 start-0 z-50 w-60 bg-[#F8F9F9] shadow-lg outline-none">
                <Dialog.Close asChild>
                  <button
                    aria-label="Close navigation"
                    className="absolute top-3 end-3 w-11 h-11 flex items-center justify-center rounded text-primary hover:bg-ocean-10"
                  >
                    <XMarkIcon className="w-6 h-6" />
                  </button>
                </Dialog.Close>
                <Sidebar {...sidebarProps} />
              </Dialog.Content>
            </Dialog.Portal>
          </Dialog.Root>
        </header>

        <main className="flex-1 min-w-0 overflow-y-auto">
          {/* 32px between page regions (topbar ↔ sections ↔ …); owned pairs (filters+table,
              header+content) go in tighter gap-2/gap-3 sub-stacks inside a region.
              Container tier comes from `size` — never hardcode a single max-w for every page. */}
          <div className={cn("mx-auto w-full p-4 md:p-6 lg:p-8 xl:p-10 flex flex-col gap-8", CONTENT_SIZES[size])}>
            {children}
          </div>
        </main>
      </div>
    </div>
  );
}
```

> Public/auth pages (login, signup, password reset) do NOT use `AppShell` — render fullscreen, form centered, `max-w-md`.

## 5. Sidebar

Current product spec (HTML prototypes, supersedes the older Figma frame in `figma-exact.md`). Same component serves desktop fixed and mobile drawer.

Canonical measurements from Balances `418:12650` (the same full-page reference frame as the
topbar and table above).

```
Width        : 293px · bg #F8F9F9 (--bg) · border-inline-end 1px border-subtle
             : sticky top-0, own scroll (max-h-screen overflow-y-auto)
Padding      : 16px inline · 24px top  (inner nav column = 261px)
Profile      : merchant block 70px tall — logo/avatar 38px radius 4 + name Bold 14px
               + merchant-ID chip (11px, cadet-base, 1px border-border, radius 4, px-1.5)
               + chevron-down at the end · 16px below it before the nav
Nav link     : 44px tall · full 261px width · flex gap-2.5 · px-3 · radius 8px · Noto Sans 14px
               4px between items (48px pitch)
Icon         : 18px Heroicons outline · stroke-width 1.5 · currentColor
Default      : text primary (#001F5F)
Active       : text cadet-base + font-semibold (no arrow icon)
Hover        : bg rgba(0,31,95,0.06)
Section label: "Manage" — Noto Sans SemiBold ~15px foreground, indented 16px, 19px block,
               8px above its first item · 24px between the previous group and the label
               (NOT the tiny 11px uppercase gray treatment — that was wrong)
```

```tsx
// components/layouts/Sidebar.tsx
import {
  HomeIcon, ArrowsUpDownIcon, LinkIcon, CurrencyDollarIcon,
  UsersIcon, ArchiveBoxIcon, DevicePhoneMobileIcon,
} from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

type NavItem = { label: string; icon: React.ElementType; href: string; active?: boolean };
type NavGroup = { label?: string; items: NavItem[] };   // ungrouped items first, then labeled sections

const NAV_GROUPS: NavGroup[] = [
  { items: [
    { label: "Home",     icon: HomeIcon,          href: "/home" },
    { label: "Payments", icon: ArrowsUpDownIcon,  href: "/payments" },
  ]},
  { label: "Payment Links", items: [
    { label: "Payment Links", icon: LinkIcon, href: "/payment-links" },
  ]},
  { label: "Manage", items: [
    { label: "Products",  icon: ArchiveBoxIcon,        href: "/products" },
    { label: "Customers", icon: UsersIcon,             href: "/customers" },
    { label: "Terminals", icon: DevicePhoneMobileIcon, href: "/terminals" },
  ]},
];

function NavButton({ item }: { item: NavItem }) {
  return (
    <a
      href={item.href}
      className={cn(
        "flex items-center gap-2.5 h-11 px-3 rounded-lg text-sm leading-snug transition-colors",
        "hover:bg-primary/5",
        item.active ? "text-cadet-base font-semibold" : "text-primary font-normal"
      )}
    >
      <item.icon className="w-[18px] h-[18px] shrink-0 [&_path]:stroke-[1.5]" />
      <span className="flex-1 truncate">{item.label}</span>
    </a>
  );
}

export function Sidebar({
  name = "Azazy",
  initials = "A",
  merchantId = "MID-14531-398",
  groups = NAV_GROUPS,
}: {
  name?: string;
  initials?: string;
  merchantId?: string;
  groups?: NavGroup[];
}) {
  return (
    <aside className="bg-[#F8F9F9] w-[293px] shrink-0 px-4 pt-6 pb-5 border-e border-border-subtle sticky top-0 max-h-screen overflow-y-auto">
      {/* Merchant block: 70px tall — logo + name + merchant-ID chip + chevron */}
      <div className="flex items-center gap-2.5 h-[70px] mb-4">
        <div className="w-[38px] h-[38px] rounded bg-primary flex items-center justify-center text-white text-xs font-bold shrink-0">
          {initials}
        </div>
        <div className="min-w-0 flex-1">
          <div className="text-sm font-bold text-foreground truncate">{name}</div>
          <span className="inline-block text-[11px] text-cadet-base border border-border rounded px-1.5">{merchantId}</span>
        </div>
        <ChevronDownIcon className="w-5 h-5 shrink-0 text-foreground" />
      </div>
      {/* 4px between items (48px pitch), 24px between groups */}
      <nav className="flex flex-col gap-1">
        {groups.map((group, gi) => (
          <div key={gi} className={cn("flex flex-col gap-1", gi > 0 && "mt-6")}>
            {group.label && (
              <p className="ms-4 mb-2 font-body text-[15px] font-semibold text-foreground">
                {group.label}
              </p>
            )}
            {group.items.map(item => <NavButton key={item.href} item={item} />)}
          </div>
        ))}
      </nav>
    </aside>
  );
}
```

Set the active item by mapping over items: `{ ...i, active: i.href === pathname }`.

## 6. PageHeader

Every page starts with one. Breadcrumb on top, then title row with right-aligned actions, then optional description. Stacks on mobile — actions drop below the title, full-width.

```tsx
// components/layouts/PageHeader.tsx
import { Breadcrumb } from "@/components/ui/breadcrumb";

export function PageHeader({
  title,
  description,
  breadcrumb,
  actions,
}: {
  title: string;
  description?: string;
  breadcrumb?: { label: string; href?: string }[];
  actions?: React.ReactNode;    // <Button>s — right-aligned desktop, stacked mobile
}) {
  return (
    <div className="flex flex-col gap-2">
      {breadcrumb && <Breadcrumb items={breadcrumb} />}
      <div className="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
        <h1 className="font-display text-h6 md:text-h3 font-bold text-foreground">{title}</h1>
        {actions && (
          <div className="flex flex-col sm:flex-row gap-2 sm:gap-2.5 [&>*]:w-full sm:[&>*]:w-auto">
            {actions}
          </div>
        )}
      </div>
      {description && (
        <p className="font-body text-xs text-muted-foreground max-w-2xl">{description}</p>
      )}
    </div>
  );
}
```

Title scale: **39px `text-h3`** on desktop (Figma topbar spec), `text-h6`–`text-h5` (20–25px) on mobile. `text-h1`/`h2`/Lead stay reserved for marketing and POS amounts.

### 6b. Desktop topbar (unified — Figma `2018:43775`/`3496:24627` + prototype locale controls)

The portal page-header row. It combines the Figma topbar with the prototype's **language + currency** controls (payment-links.html), grouped by the **law of proximity** — gaps encode relationship, so within-group gaps are always visibly smaller than between-group gaps:

```
[Title 39px · flex-1] [Module search 370px] ‖ [عربي · EGP▾] [ModeChip] [? 🔔 ⚙] [＋]
```

```
GROUPING & GAPS (proximity: distance = relatedness) — canonical Balances `418:13083`
  inside a control      : 4–8px   (chip icon↔label, currency↔chevron)
  within a group        : 16px    (utility icons: 24px icons on a 40px pitch)
  between groups        : 32px    (Developers ‖ ModeChip ‖ utilities ‖ create — measured
                          consistently at 32px in the reference frame, not 24px)
  title↔search↔controls : title takes flex-1; search ends the start group

Row       : height 47px · width = the content container (1305 canonical)
Title     : Outfit Bold 39px (text-h3) #202020 · flex-1
Search    : 370×40 (shrink-0) · **rounded-full pill** · bg #F8F8F8 · 1px border
            text inset 16px · 24px search icon 16px from the end
            placeholder Noto Sans Regular 16px #919C9C
            — searches the CURRENT MODULE, and it is THE page search: never render a
              second search in the table toolbar or elsewhere on the same page
Developers: text button Noto Sans SemiBold 14px primary · 105×33 · radius 5
            — PRESENT in the canonical frame; render it for merchant portal pages
Mode chip : ModeChip (components.md) — test/live states · 142×38
Utilities : 24px question-mark-circle (outline) · bell (outline) ·
            **cog-6-tooth (heroicons-SOLID)** — the cog is the filled variant, the other
            two are outline · 16px gaps
Create    : 40px heroicons-solid/plus-circle · primary (navy disc, white plus)
Locale    : language + currency (عربي / EGP) are a PROTOTYPE-only addition
            (payment-links.html) — not in the canonical Figma topbar. Include them only
            when the page actually needs locale switching; they fold into the cog menu first.

Mode chip : p-2 radius 4 gap-4 · label Regular 16px + SwitchEnhanced (components.md)
            test  = bg #FFF7E8 (Warning 09) · border #FFE6B6 (Warning 08) · "Test mode"
            live  = bg #F3FFF4 (Success 09) · border #4AAB4E (Success Base) · "Live mode"

ADAPTIVE BEHAVIOR — every fixed-width piece has a defined smaller-screen fallback
(never let the row overflow or squash); Hick's law: fewer visible controls as space shrinks:
  ≥xl (1280)  : full spec — all groups, 32px between groups, 370px search
  lg–xl       : 24px between groups · search ~280px ·
                Developers (and locale, if present) fold into the settings menu (cog)
  md–lg       : title 31px · search collapses to a 24px icon button
  <md (mobile): ONLY the title renders (text-h5) — the AppShell mobile top bar is the
                page's chrome; ALL topbar controls (search, Developers, mode, utilities,
                create) are hidden `<md` IN CODE, not just in spirit: mode/locale live
                behind the drawer menu, search is the in-page gray field (lists.md §2),
                create is the page's full-width CTA. Rendering the desktop control row
                on a phone produces a double header and overlapping controls.

TOPBAR ↔ PAGE : the topbar is a REGION — 32px of space below it before the first
                page section (larger than any gap inside the page body)
```

```tsx
// components/layouts/Topbar.tsx
import { MagnifyingGlassIcon, QuestionMarkCircleIcon, BellIcon } from "@heroicons/react/24/outline";
import { PlusCircleIcon, Cog6ToothIcon } from "@heroicons/react/24/solid";   // cog + create are SOLID

export function Topbar({ title, searchPlaceholder, onSearch, mode, onModeChange, onCreate,
  onDevelopers, locale }: {
  title: string;
  searchPlaceholder: string;         // "Find an account..."
  onSearch: (q: string) => void;
  mode: "test" | "live";
  onModeChange: (m: "test" | "live") => void;
  onCreate: () => void;
  onDevelopers?: () => void;         // canonical frame renders this — omit only if the product has no dev tools
  locale?: React.ReactNode;          // prototype-only عربي/EGP controls; not in the canonical Figma topbar
}) {
  return (
    // Proximity: 32px BETWEEN groups (canonical); smaller gaps INSIDE groups.
    // Adaptive: gaps, search width, and Developers all step down before anything overflows.
    <div className="flex items-center gap-6 xl:gap-8 h-[47px]">
      <div className="flex flex-1 items-center gap-6 min-w-0">
        {/* Title takes the stretch (flex-1) so the search sits at the END of this group. */}
        <h1 className="flex-1 font-display text-h5 md:text-h4 xl:text-h3 font-bold text-foreground truncate">{title}</h1>
        {/* Full search ≥lg (280→370px) — PILL shaped, 24px icon 16px from the end */}
        <div className="hidden lg:flex shrink-0 items-center justify-between w-[280px] xl:w-[370px] h-10 rounded-full bg-[#F8F8F8] border border-[#F3F3F3] ps-4 pe-4">
          <input placeholder={searchPlaceholder} onChange={e => onSearch(e.target.value)}
            className="flex-1 min-w-0 bg-transparent border-0 font-body text-base text-foreground placeholder:text-cadet-05 focus:outline-none" />
          <MagnifyingGlassIcon className="w-6 h-6 text-foreground shrink-0" />
        </div>
        <button aria-label="Search" className="hidden md:flex lg:hidden w-10 h-10 items-center justify-center rounded text-cadet-base hover:bg-muted">
          <MagnifyingGlassIcon className="w-6 h-6" />
        </button>
      </div>

      {/* ENTIRE control region hidden <md — on phones the AppShell bar + drawer own these */}
      <div className="hidden md:flex items-center gap-6 xl:gap-8 shrink-0">
        {/* Developers — present in the canonical frame; folds into the cog menu <xl */}
        {onDevelopers && (
          <button onClick={onDevelopers}
            className="hidden xl:block font-body text-xs font-semibold text-primary px-3.5 py-2 rounded-[5px] hover:bg-ocean-10">
            Developers
          </button>
        )}

        {/* Prototype-only locale controls, when a page genuinely needs them */}
        {locale && <div className="hidden xl:flex items-center gap-3">{locale}</div>}

        <ModeChip mode={mode} onChange={onModeChange} />   {/* components.md */}

        {/* Utility cluster (16px inside) — cog is the SOLID variant, the other two outline */}
        <div className="flex items-center gap-4 text-foreground">
          <QuestionMarkCircleIcon className="w-6 h-6" />
          <BellIcon className="w-6 h-6" />
          <Cog6ToothIcon className="w-6 h-6" />
        </div>

        <button aria-label="Create" onClick={onCreate} className="text-primary">
          <PlusCircleIcon className="w-10 h-10" />   {/* @heroicons/react/24/solid */}
        </button>
      </div>
    </div>
  );
}
```

## 7. Page Templates

Pick the template first, then fill the slots. These four cover ~95% of Kashier screens.

### 7a. List page (transactions, customers, payouts…)

`AppShell` defaults to `size="wide"` (fluid, caps at 1840px — layout.md §2), which is what a list
page wants. Row click opens a `DetailDrawer` in place — it does NOT navigate to a separate detail
route unless the record is complex enough to need its own URL (`patterns.md` §1).

```tsx
<AppShell>
  {/* Topbar carries the title AND the page's only search (§6b) */}
  <Topbar title="Payments" searchPlaceholder="Find a payment" mode={mode} onModeChange={setMode} onCreate={…} />

  {/* Primary CTA gets its OWN end-aligned row above the filters (canonical 418:12514) */}
  <div className="flex justify-end">
    <Button onClick={() => setCreateOpen(true)}>
      <PlusIcon className="w-[18px] h-[18px]" />New Account
    </Button>
  </div>

  {/* Optional KPI row */}
  <KpiBar items={kpis} />

  {/* Filters OWN the table → both live in one tight sub-stack, while the stack itself
      sits 32px from its neighbors (page gap-8). Export is the END of the FILTER row —
      NOT a toolbar inside the table card (tables.md §1). */}
  <div className="flex flex-col gap-3">
    <div className="flex items-center gap-4 flex-wrap">
      <FilterChip icon={CheckCircleIcon} label="Select a method" />
      <FilterChip icon={TagIcon} label="Select a label" />
      <FilterChip icon={CalendarDaysIcon} label="Select a value date range" />
      <button className="ms-auto flex items-center gap-1.5 font-body text-xs font-semibold text-primary">
        <ArrowUpTrayIcon className="w-[18px] h-[18px]" />Export
      </button>
    </div>

    {/* Data: table ≥md, list <md — see tables.md §5. No toolbar, no second search. */}
    <DataTable
      columns={columns}
      rows={pageRows}
      pagination={paginationProps}
      onRowClick={row => setDetailRow(row)}   {/* opens DetailDrawer — patterns.md §3a */}
    />
  </div>

  <DetailDrawer open={!!detailRow} onOpenChange={() => setDetailRow(null)} row={detailRow} />
  <TaskDrawer open={importOpen} onOpenChange={setImportOpen} />
</AppShell>
```

### 7b. Detail page (transaction detail, customer detail…)

Use a full detail **page** (this template) only when the record needs its own URL/deep link or has
more content than a drawer body comfortably holds (§1's "full page" rung). Otherwise the same
content fits a `DetailDrawer` launched from the list (§7a) — don't build both. When it is a page,
use `size="detail"` (caps at 1440px — a 1840px-wide 2-column detail grid reads worse than a 480px
drawer would have).

```tsx
<AppShell size="detail">
  <PageHeader
    title="TXN-98765"
    breadcrumb={[{ label: "Payments", href: "/payments" }, { label: "TXN-98765" }]}
    actions={<>
      <Button variant="outline">Refund</Button>
      <Button>Download receipt</Button>
    </>}
  />

  {/* 2/3 main + 1/3 summary sidebar on lg; stacked below */}
  <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 items-start">
    <div className="lg:col-span-2 flex flex-col gap-6 min-w-0">
      <Card>…primary content…</Card>
      <Card>…activity / timeline…</Card>
    </div>
    <div className="flex flex-col gap-6 min-w-0">
      <Card>…summary (CardRow label/value pairs)…</Card>
      <Card>…customer info…</Card>
    </div>
  </div>
</AppShell>
```

### 7c. Form page (create payment link, settings…)

`size="form"` makes `AppShell` itself cap at 672px — don't ALSO put `max-w-2xl` on the `<form>`
(that's a double constraint, and the two numbers can drift apart over time).

```tsx
<AppShell size="form">
  <PageHeader title="New payment link" breadcrumb={…} />

  <form noValidate onSubmit={handleSubmit} className="flex flex-col gap-6">
    <Card>
      <CardInner>
        <CardHeader><CardTitle>Details</CardTitle></CardHeader>
        <div className="flex flex-col gap-4">
          <FormField label="Amount" required error={!!errors.amount} errorMessage={errors.amount}>
            <Input type="number" placeholder="0.00" error={!!errors.amount} />
          </FormField>
          {/* Two short fields may share a row ≥sm; never 3+ columns in forms */}
          <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <FormField label="Currency"><Select /></FormField>
            <FormField label="Expiry"><Input type="date" /></FormField>
          </div>
        </div>
      </CardInner>
    </Card>

    {/* Sticky footer actions */}
    <div className="sticky bottom-0 -mx-4 md:-mx-6 lg:mx-0 px-4 md:px-6 lg:px-0 py-3 bg-white border-t border-border-subtle lg:border-0 lg:bg-transparent lg:py-0 flex flex-col-reverse sm:flex-row sm:justify-end gap-2">
      <Button type="button" variant="ghost" className="w-full sm:w-auto">Cancel</Button>
      <Button type="submit" className="w-full sm:w-auto">Create link</Button>
    </div>
  </form>
</AppShell>
```

### 7d. Dashboard / home

```tsx
<AppShell>
  <PageHeader title="Good morning, Misr Elkheir" description="Here's what's happening today." />

  {/* KPI row: 2-up mobile, 4-up desktop */}
  <KpiBar items={kpis} />

  {/* Card grid: 1 → 2 → 3 columns */}
  <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
    <Card variant="raised">…chart / widget…</Card>
    <Card variant="raised">…recent activity list…</Card>
    <Card variant="raised">…quick actions…</Card>
  </div>
</AppShell>
```

### 7d-bis. Balances / stats page (Figma Balances `10:2413`)

The pattern for account-scoped stats pages: topbar → end-aligned primary action → account cards → tabs → KPI row → titled table sections.

```tsx
<AppShell>
  <Topbar title="Balances" searchPlaceholder="Find a transaction" … />
  <div className="flex justify-end">
    <Button><PencilIcon className="w-[18px] h-[18px]" />Manage accounts</Button>
  </div>
  <AccountSelector accounts={accounts} selectedId={sel} onSelect={setSel} />   {/* components.md */}
  <Tabs tabs={[{label:"Overview"},{label:"Top-ups"},{label:"Activities"}]} />
  <KpiBar items={[{label:"Available balance",value:"1M EGP"}, …]} />

  {/* Titled section: header OWNS the table → gap-2 inside; 32px to other sections */}
  <section className="flex flex-col gap-2">
    <div>
      <h2 className="font-body text-base font-bold text-foreground">Settlement forecast</h2>
      <p className="font-body text-xs text-muted-foreground">Description</p>
    </div>
    <DataTable columns={[
      { key:"date", header:"Date" },
      { key:"amount", header:"To be received", align:"end", render:r => <CurrencyCell amount={r.amount} /> },
    ]} rows={forecast} />
  </section>
  <section>…Recent transfers (same shape)…</section>
</AppShell>
```

### 7e. Mobile list page (Customers mobile — Figma `455:9921`)

The `<lg` rendering of a records page. Everything stacks, CTA goes full-width, records render as `RecordGroup`s (see `lists.md` §1):

```tsx
<AppShell>{/* mobile top bar: account switcher · bell · menu */}
  <h1 className="font-display text-h5 font-bold text-foreground">Customers</h1>
  <Button className="w-full h-12 text-base font-semibold"><PlusIcon className="w-6 h-6" />New Customer</Button>

  {/* Soft-gray search (not the bordered Input) + dashed date chip + select-all — lists.md §2 */}
  <SearchField placeholder="Find a customer" />
  <div className="flex items-center justify-between">
    <DateRangeChip label="Select a date range" />
    <SelectAllButton />
  </div>

  <RecordGroup title="Today">…RecordCards…</RecordGroup>
  <RecordGroup title="Yesterday">…RecordCards…</RecordGroup>
</AppShell>
```

## 8. Grid Recipes — quick reference

| Content | Classes |
|---------|---------|
| KPI stats | `grid grid-cols-2 md:grid-cols-4 gap-px` (or `KpiBar`) |
| Card grid | `grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6` |
| Detail 2/3 + 1/3 | `grid grid-cols-1 lg:grid-cols-3 gap-6 items-start` + `lg:col-span-2` |
| Form field pair | `grid grid-cols-1 sm:grid-cols-2 gap-4` |
| Filter toolbar | `flex flex-col sm:flex-row gap-2` with `flex-1` search |
| Centered auth card | `min-h-screen flex items-center justify-center p-4` + `w-full max-w-md` |

**Grid gotchas:** put `min-w-0` on grid/flex children that contain tables or truncating text (prevents blowout); use `items-start` on detail grids so the sidebar column doesn't stretch; never build layouts with fixed pixel widths outside the shell — use fractions and max-widths.

## 9. Overlays — Drawers & Modals

Page templates above cover what's IN the shell. For anything that floats ABOVE it — record detail,
a multi-step task, a confirmation — load `patterns.md` first:

- **§1 Disclosure ladder** — decide inline vs. drawer vs. modal vs. full page before picking a
  template from §7 at all. A drawer is the default overlay; a modal is the exception.
- **§2 Feature placement IA** — decide whether a new feature even needs a page/nav entry, or
  whether it's a drawer-only task launched from an existing screen (most "add X" / "bulk Y" flows
  are the latter — see the worked examples in `patterns.md` §2).
- **§3 Drawer component** — `DetailDrawer` (480px, record view) and `TaskDrawer` (620px,
  multi-step/selection), both end-anchored, both collapsing to a bottom sheet `<md`.
- **§4–5 Motion, focus, a11y, destructive actions** — required for every overlay, not optional
  polish.

§7a and §7b above show a `DetailDrawer`/`TaskDrawer` wired into the list and detail templates —
read those alongside `patterns.md` before building either.
