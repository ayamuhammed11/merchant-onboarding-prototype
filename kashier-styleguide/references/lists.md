# Kashier Lists — RecordCard (entity lists), ListView (menus), Patterns

> Load this file for any vertical list. **Not all lists are tables** — and in this DS, entity lists
> are not divider rows either: they are **record cards**.
>
> - Records (customers, payment links, orders…) → **RecordCard** (§1) — canonical, from Figma
> - Menus, settings, pickers, bottom sheets → **ListView/ListItem** (§4)
> - POS transaction lists → shadowed POS cards (`platforms.md` §4b)

## 1. RecordCard — canonical entity list

Source: Payment Links card `2237:44002` (file `Kw8SaHwCXqMlu6AbyRMeNq`) · Customers record `506:9868`, mobile `455:9921` (file `4LRW6jrxg8sgSIlgtJVN2S`). Same component serves desktop AND mobile — responsiveness comes from `flex-wrap` + min-widths, not breakpoint forks.

```
Card      : bg white · border 1px border-subtle (#F5F6F6) · radius 4 · p-4 (16px)
Stack     : 16px gap between cards (8px inside a mobile group)
Grouping  : by day — "Today"/"Yesterday" headers · Outfit Bold 25px desktop / 20px mobile,
            black · 24px between groups · 16px (desktop) / 8px (mobile) header→cards
Primary row (flex flex-wrap gap-2):
  Identity : 18px Heroicon + title Noto Sans SemiBold 14px #202020
             + ref 12px #839090 (2px gap) · fixed w-[300px] min-w-[250px]
  Attribute: 18px icon + SemiBold 12px cadet-base · flex-1 min-w-[150px] max-w-[310px]
  Chips    : LabelChip pills (see below), end side on desktop / bottom of card on mobile
  Kebab    : absolute top-end (-12px offset) · 39px hit area · 19px ellipsis-vertical
             · radius-40 hover bg
Divider   : border-t border-subtle · pt-2
Meta row (flex flex-wrap gap-2):
  Item     : label Regular 12px #839090 over value SemiBold 12px cadet-base (2px gap)
             · flex-1 min-w-[150px]  ← min-width collapses 4 cols → 2 cols on mobile
Hover     : select checkbox becomes visible (Figma annotation) — pointer devices only
Checkbox  : 16px · radius 4 · border #D5D9D9 · checked = navy #001F5F border + navy check
            (outline style — NOT filled)
```

```tsx
// components/ui/record-card.tsx
import { EllipsisVerticalIcon } from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

// Day group: header + card stack
export function RecordGroup({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="flex flex-col gap-2 md:gap-4">
      <h2 className="font-display text-h6 md:text-h5 font-bold text-foreground">{title}</h2>
      <div className="flex flex-col gap-2 md:gap-4">{children}</div>
    </section>
  );
}

export function RecordCard({ children, onSelectChange, selected, className }: {
  children: React.ReactNode;
  selected?: boolean;
  onSelectChange?: (v: boolean) => void;   // enables the hover-reveal checkbox
  className?: string;
}) {
  return (
    <div className={cn(
      "group relative bg-white border border-border-subtle rounded p-4 flex flex-col gap-4",
      selected && "border-primary",
      className
    )}>
      {onSelectChange && (
        /* Hover-reveal select box (Figma annotation). Always visible once selected. */
        <input
          type="checkbox"
          checked={selected}
          onChange={e => onSelectChange(e.target.checked)}
          className={cn(
            "absolute top-4 -start-0.5 w-4 h-4 rounded border border-cadet-07 accent-primary transition-opacity",
            selected ? "opacity-100" : "opacity-0 group-hover:opacity-100 group-focus-within:opacity-100"
          )}
        />
      )}
      {children}
    </div>
  );
}

// Primary row — identity + attributes + chips wrap naturally
export function RecordRow({ children }: { children: React.ReactNode }) {
  return <div className="flex flex-wrap gap-2 items-start w-full">{children}</div>;
}

// Identity cell: icon + title + reference
export function RecordIdentity({ icon: I, title, subtitle }: {
  icon: React.ComponentType<{ className?: string }>;
  title: string;
  subtitle?: string;
}) {
  return (
    <div className="flex gap-1 items-start w-[300px] min-w-[250px] max-w-full">
      <I className="w-[18px] h-[18px] shrink-0 text-foreground" />
      <div className="flex flex-col gap-0.5 min-w-0">
        <p className="font-body text-xs font-semibold text-foreground">{title}</p>
        {subtitle && <p className="font-body text-xxs text-cadet-04">{subtitle}</p>}
      </div>
    </div>
  );
}

// Attribute cell: icon + value (email, phone, …)
export function RecordAttribute({ icon: I, children }: {
  icon: React.ComponentType<{ className?: string }>;
  children: React.ReactNode;
}) {
  return (
    <div className="flex flex-1 gap-1 items-center min-w-[150px] max-w-[310px]">
      <I className="w-[18px] h-[18px] shrink-0 text-foreground" />
      <span className="font-body text-xxs font-semibold text-cadet-base truncate">{children}</span>
    </div>
  );
}

// Meta footer: divider + wrapping label/value items
export function RecordMeta({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex flex-wrap gap-2 items-start w-full border-t border-border-subtle pt-2">
      {children}
    </div>
  );
}
export function RecordMetaItem({ label, value }: { label: string; value: React.ReactNode }) {
  return (
    <div className="flex flex-1 flex-col gap-0.5 min-w-[150px]">
      <span className="font-body text-xxs text-cadet-04">{label}</span>
      <span className="font-body text-xxs font-semibold text-cadet-base">{value}</span>
    </div>
  );
}

// Label chip: pill, Neutral 09 bg + Neutral 08 border
export function LabelChip({ children }: { children: React.ReactNode }) {
  return (
    <span className="inline-flex items-center rounded-2xl bg-muted border border-border ps-2 pe-1 py-0.5 font-body text-xs text-foreground whitespace-nowrap">
      {children}
    </span>
  );
}

// Kebab: 39px hit area pinned to the card's top-end corner
export function EllipsisMenuButton(props: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  return (
    <button
      aria-label="More actions"
      className="absolute top-1 end-1 w-[39px] h-[39px] flex items-center justify-center rounded-full text-cadet-base hover:bg-muted"
      {...props}
    >
      <EllipsisVerticalIcon className="w-[19px] h-[19px]" />
    </button>
  );
}
```

### Usage — Customers list (desktop & mobile, same code)

```tsx
<div className="flex flex-col gap-6">
  <RecordGroup title="Today">
    {customers.map(c => (
      <RecordCard key={c.id} selected={sel.has(c.id)} onSelectChange={v => toggle(c.id, v)}>
        <EllipsisMenuButton onClick={() => openMenu(c)} />
        <RecordRow>
          <RecordIdentity icon={UserIcon} title={c.name} subtitle={c.ref} />
          <RecordAttribute icon={AtSymbolIcon}>{c.email}</RecordAttribute>
          <RecordAttribute icon={DevicePhoneMobileIcon}>{c.phone}</RecordAttribute>
          <div className="hidden md:flex gap-2.5 items-center">
            {c.labels.map(l => <LabelChip key={l}>{l}</LabelChip>)}
          </div>
        </RecordRow>
        <RecordMeta>
          <RecordMetaItem label="Age" value={c.age} />
          <RecordMetaItem label="Occupation" value={c.occupation} />
          <RecordMetaItem label="Preferred Channel" value={c.channels} />
          <RecordMetaItem label="Preferred language" value={c.language} />
        </RecordMeta>
        {/* Mobile: chips move to the bottom of the card */}
        <div className="flex md:hidden gap-2.5 items-center">
          {c.labels.map(l => <LabelChip key={l}>{l}</LabelChip>)}
        </div>
      </RecordCard>
    ))}
  </RecordGroup>
  <RecordGroup title="Yesterday">…</RecordGroup>
</div>
```

### Payment-links variant

Identity gets the larger style (title SemiBold 16px `text-cadet-base`, ref 14px `text-cadet-04`), the amount + status sit inline in the primary row, and meta items are horizontal icon+text:

```tsx
<RecordRow>
  <div className="flex flex-col gap-0.5 w-[220px] min-w-[180px]">
    <p className="font-body text-base font-semibold text-cadet-base leading-none">{link.title}</p>
    <p className="font-body text-xs text-cadet-04">{link.ref}</p>
  </div>
  <div className="flex gap-2 items-center flex-1 min-w-[150px] ps-6">
    <span className="font-body text-base font-bold text-foreground">{link.amount}</span>
    <StatusChip tone="positive">Paid</StatusChip>   {/* from tables.md */}
  </div>
  <div className="flex gap-2.5 items-center justify-end pe-8">
    {link.labels.map(l => <LabelChip key={l}>{l}</LabelChip>)}
  </div>
</RecordRow>
<RecordMeta>
  {/* horizontal icon + text meta items */}
  <div className="flex gap-1 items-center min-w-[150px]">
    <CalendarDaysIcon className="w-4 h-4 text-cadet-04" />
    <span className="font-body text-xs text-cadet-04">Due {link.due}</span>
    {link.expired && <StatusChip tone="neutral">Expired</StatusChip>}
  </div>
  <div className="flex flex-1 gap-1 items-center min-w-[150px]">
    <HashtagIcon className="w-4 h-4 text-cadet-04" />
    <span className="font-body text-xs text-cadet-04">{link.reference}</span>
  </div>
</RecordMeta>
```

## 2. List page furniture (mobile — Customers `455:9921`)

Above the record groups on mobile: title Outfit Bold 25px → full-width primary Button (h-12, 24px plus icon, SemiBold 16px label) → search field → dashed date-range chip → "Select all".

```tsx
{/* Search field — soft gray, not the bordered Input */}
<div className="flex items-center justify-between w-full rounded bg-[#F8F8F8] border border-[#F3F3F3] ps-4 pe-2 py-2">
  <input placeholder="Find a customer"
    className="flex-1 bg-transparent border-0 font-body text-xs text-cadet-base placeholder:text-cadet-base focus:outline-none" />
  <MagnifyingGlassIcon className="w-6 h-6 text-cadet-base" />
</div>

{/* Date-range chip — dashed border */}
<button className="flex items-center gap-2 rounded border border-dashed border-cadet-07 p-2 font-body text-xxs text-cadet-base">
  <CalendarDaysIcon className="w-[18px] h-[18px]" /> Select a date range
</button>

{/* Select all — text button */}
<button className="flex items-center gap-1 px-3 py-2 rounded-sm font-body text-xxs font-semibold text-primary">
  <CheckIcon className="w-3.5 h-3.5" /> Select all
</button>
```

## 2b. PickerRow (multi-select rows inside a TaskDrawer)

Source: Add-Customers task drawer (`patterns.md` §3b, Figma `77:1403`). Same identity/meta
building blocks as `RecordCard` (icon+title+ref, tag chips) — reused rather than invented fresh, so
a picker row and a record card read as the same "entity" language (Similarity law).

```
Row     : h-[68px] · px-6 py-2.5 · hover bg-muted · border-b border-border-subtle (last:none)
Checkbox: 15px, start-aligned
Avatar  : 34px rounded-full, initials, 12px gap to checkbox and to content
Content : name 14px semibold → meta 12px muted (phone · email · #ID) → tag chips (LabelChip, 4px gap)
```

```tsx
// components/ui/picker-row.tsx
import { LabelChip } from "./record-card";   // reuse — lists.md §1
import { cn } from "@/lib/utils";

export function PickerRow({ id, name, meta, tags = [], selected, onToggle }: {
  id: string; name: string; meta: string; tags?: string[];
  selected: boolean; onToggle: (id: string) => void;
}) {
  const initials = name.split(" ").map(w => w[0]).slice(0, 2).join("").toUpperCase();
  return (
    <label className={cn(
      "flex items-center gap-3 h-[68px] px-2 border-b border-border-subtle last:border-0 cursor-pointer",
      "hover:bg-muted transition-colors",
      selected && "bg-ocean-10 hover:bg-ocean-10"
    )}>
      <input type="checkbox" checked={selected} onChange={() => onToggle(id)}
        className="w-[15px] h-[15px] rounded border-border shrink-0" />
      <div className="w-[34px] h-[34px] rounded-full bg-primary text-white flex items-center justify-center text-xs font-bold shrink-0">
        {initials}
      </div>
      <div className="min-w-0 flex-1 flex flex-col gap-0.5">
        <p className="font-body text-xs font-semibold text-foreground truncate">{name}</p>
        <p className="font-body text-xxs text-muted-foreground truncate">{meta}</p>
        {tags.length > 0 && (
          <div className="flex gap-1 flex-wrap mt-0.5">
            {tags.map(t => <LabelChip key={t}>{t}</LabelChip>)}
          </div>
        )}
      </div>
    </label>
  );
}
```

Used exclusively inside a `TaskDrawer`'s scrollable body (`patterns.md` §3b) — not a general-purpose
row; for standalone entity lists on a page, use `RecordCard` (§1) instead.

## 3. ListView + ListItem (menus, settings, pickers, bottom sheets ONLY)

> Do NOT use for entity lists — those are RecordCards (§1). This divider-row list is for
> navigation menus, settings screens, selectable pickers, and action sheets.

```tsx
// components/ui/list.tsx
import { ChevronRightIcon } from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

export function ListView({
  children,
  loading = false,
  emptyState,
  footer,
  className,
}: {
  children?: React.ReactNode;
  loading?: boolean;
  emptyState?: React.ReactNode;      // <EmptyState …/> from tables.md
  footer?: React.ReactNode;
  className?: string;
}) {
  const isEmpty = !loading && React.Children.count(children) === 0;
  return (
    <div className={cn("bg-white border border-border-subtle rounded overflow-hidden", className)}>
      <div className="divide-y divide-border-subtle">
        {loading ? <ListSkeleton /> : isEmpty ? emptyState : children}
      </div>
      {footer && !isEmpty && !loading && (
        <div className="border-t border-border-subtle">{footer}</div>
      )}
    </div>
  );
}

export function ListItem({
  leading,
  title,
  subtitle,
  trailing,
  onPress,
  href,
  selected = false,
  chevron = false,
  className,
}: {
  leading?: React.ReactNode;         // <Avatar> or <ListItemIcon>
  title: React.ReactNode;
  subtitle?: React.ReactNode;
  trailing?: React.ReactNode;        // value, <Switch>, or nothing
  onPress?: () => void;
  href?: string;
  selected?: boolean;
  chevron?: boolean;
  className?: string;
}) {
  const interactive = Boolean(onPress || href);
  const Tag: any = href ? "a" : interactive ? "button" : "div";
  return (
    <Tag
      href={href}
      onClick={onPress}
      aria-selected={selected || undefined}
      className={cn(
        "w-full flex items-center gap-3 px-4 py-3 min-h-14 text-start font-body transition-colors",
        interactive && "cursor-pointer hover:bg-muted active:bg-cadet-09",
        selected && "bg-ocean-10 hover:bg-ocean-10",
        className
      )}
    >
      {leading && <div className="shrink-0">{leading}</div>}
      <div className="flex-1 min-w-0 flex flex-col gap-0.5">
        <span className="text-sm font-semibold text-foreground truncate">{title}</span>
        {subtitle && <span className="text-xxs text-muted-foreground truncate">{subtitle}</span>}
      </div>
      {trailing && <div className="shrink-0 text-end">{trailing}</div>}
      {chevron && <ChevronRightIcon className="w-4 h-4 shrink-0 text-cadet-05 rtl:rotate-180" />}
    </Tag>
  );
}

// Icon leading slot: 40px muted circle
export function ListItemIcon({ icon: Icon }: { icon: React.ComponentType<{ className?: string }> }) {
  return (
    <div className="w-10 h-10 rounded-full bg-muted flex items-center justify-center">
      <Icon className="w-5 h-5 text-cadet-base" />
    </div>
  );
}

function ListSkeleton({ count = 5 }: { count?: number }) {
  return (
    <>
      {Array.from({ length: count }).map((_, i) => (
        <div key={i} className="flex items-center gap-3 px-4 py-3 min-h-14">
          <div className="w-10 h-10 rounded-full bg-cadet-09 animate-pulse shrink-0" />
          <div className="flex-1 flex flex-col gap-1.5">
            <div className="h-3.5 w-2/5 rounded bg-cadet-09 animate-pulse" />
            <div className="h-2.5 w-3/5 rounded bg-cadet-09 animate-pulse" />
          </div>
          <div className="h-3.5 w-16 rounded bg-cadet-09 animate-pulse" />
        </div>
      ))}
    </>
  );
}
```

### Menu / settings / picker patterns

```tsx
{/* Action list (settings, sheets) */}
<ListView>
  <ListItem leading={<ListItemIcon icon={UserIcon} />} title="Profile" chevron onPress={…} />
  <ListItem leading={<ListItemIcon icon={BellIcon} />} title="Notifications" subtitle="Email, SMS" chevron onPress={…} />
  <ListItem leading={<ListItemIcon icon={MoonIcon} />} title="Dark mode" trailing={<Switch />} />
  <ListItem leading={<ListItemIcon icon={ArrowRightStartOnRectangleIcon} />}
    title={<span className="text-destructive">Log out</span>} onPress={…} />
</ListView>

{/* Selectable picker */}
<ListView>
  {accounts.map(a => (
    <ListItem key={a.id} leading={<Checkbox checked={selected.has(a.id)} />}
      title={a.name} subtitle={a.iban} selected={selected.has(a.id)} onPress={() => toggle(a.id)} />
  ))}
</ListView>
```

For grouped menu lists, `ListSectionHeader`:

```tsx
export function ListSectionHeader({ children }: { children: React.ReactNode }) {
  return (
    <div className="px-4 py-2 bg-muted/60 text-[12px] font-semibold text-cadet-base uppercase tracking-[0.96px]">
      {children}
    </div>
  );
}
```

## 4. Description list (label/value detail data)

For read-only detail data, reuse `CardRow` inside a `Card` (see `components.md` §5) — or `RecordMetaItem`s in a wrap row for the RecordCard style.

## 5. List Checklist

- [ ] Entity list? → RecordCard with day grouping — not a table, not divider rows
- [ ] Empty state when the list can be empty (EmptyState from tables.md)
- [ ] Loading skeleton
- [ ] Kebab menu on cards (39px hit area) for row actions; hover-reveal checkbox when bulk select exists
- [ ] Meta items use `min-w-[150px]` so columns collapse naturally on narrow screens
- [ ] Interactive rows render as `<a>`/`<button>`, not clickable `<div>`s
- [ ] `truncate` + `min-w-0` on text so long values never break the layout
