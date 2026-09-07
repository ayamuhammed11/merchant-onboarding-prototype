# Kashier Patterns — Disclosure, Feature Placement, Drawers, Motion & A11y

> Load this file BEFORE building any overlay (drawer/modal/sheet) or ANY new feature surface
> (nav item, tab, settings section, topbar utility). It is the "where does this go, and what
> container carries it" layer that sits above `layout.md`'s page templates.
>
> Sources: Customer Portal — Circle-pay (`8Xb0ZVxoTa5ctB77GGdR85`) — task drawer `77:1327`→`77:1403`
> (620px, "Add Customers to Portal") and detail drawer `1291:3325`→`1290:107262` (480px,
> "Transaction Details"). Pixel specs transcribed in `figma-exact.md` under **Drawers**.

## 1. The Disclosure Ladder (routing rule for ANY new content)

Before reaching for a component, decide the *container* by walking this ladder top-down and
stopping at the first rung that fits. **Within a logical context, a drawer is the default
overlay and a modal is the exception** — treat that as a hard rule, not a preference.

| Rung | Use when | Surface |
|---|---|---|
| **Inline** | ≤2 fields, reversible, no context switch needed | expand-in-place, inline edit, popover |
| **Drawer** | record detail · a multi-field/multi-step task · browsing or selecting from a list · the user benefits from seeing the page behind it | end-anchored panel (§3) |
| **Modal** | destructive confirmation · ≤3 fields, single narrow purpose · a blocking system message | centered card (`components.md` §9c) |
| **Full page** | >1 screen of form · needs its own URL/deep link · a multi-entity workflow | route |

**Why drawers outrank modals here:** a modal traps focus in a small centered box and hides the
page; a drawer keeps the list/table/record the user was just looking at in view, scrolls its own
body independently, and can hold a persistent footer for multi-step or multi-select tasks. Both
Figma references are exactly this: "Add Customers to Portal" is a multi-step selection task
(drawer, not modal), "Transaction Details" is a record detail view (drawer, not a navigation to a
new page or a modal fighting for space with a 480px card).

The one place a modal outranks a drawer: an **irreversible action needs a blocking confirm** — see
§5.

## 2. Feature Placement IA (where does a new feature live?)

Not every feature is a sidebar entry. Before adding a nav item, score the candidate against this
ranked table and pick the *first* surface that fits — never default to primary nav because it's
the most visible option (that's what erodes Hick's law: nav must stay ≤5 groups).

| Rank | Surface | Fits when | Example |
|---|---|---|---|
| 1 | **Primary nav item** | Used ~daily, is a standalone concept, deserves its own URL/breadcrumb root | Payments, Customers, Terminals |
| 2 | **Module sub-tab** | Belongs to an existing module's noun, doesn't need its own nav slot | "Payouts" tab inside Balances |
| 3 | **Settings section** | Configure-once, admin-scoped, not part of the daily workflow | Webhook config, team roles, branding |
| 4 | **Topbar utility** | Global, cross-module, must always be reachable | Help, notifications, mode switch |
| 5 | **Row / detail action** | Acts on exactly one record | Refund, resend receipt, clone link |
| 6 | **Drawer/modal-only task** | Transient, no standing "home" of its own | Bulk import, add-to-portal, quick filters |

**Rules:**
- Score by **frequency** (daily vs. rare) × **scope** (global vs. one-record) × **ownership**
  (does it belong to an existing module?) × **URL-worthiness** (would a user bookmark or share it?).
- **Never add a nav item without running this table.** A feature that scores low on frequency and
  URL-worthiness (e.g. "add customers to a portal") belongs at rank 6, launched from wherever the
  user already is — not a new sidebar entry.
- If the top two ranks score within one point of each other, that's a genuine judgment call —
  present the options to the user (`AskUserQuestion`) before building, rather than picking silently.
- State the chosen rank and why in one line before writing markup — this is the artifact that
  proves the table was actually run, not skipped.

**Worked example — "bulk import customers":** Daily? No (occasional admin task). Standalone
concept? No — it's an operation *on* the Customers module. Global? No, scoped to one module.
URL-worthy? No, it's a transient task. → **Rank 6**: a "Bulk import" button in the Customers
module's `TableToolbar` (`tables.md` §1) opens a **task drawer** (§3), not a nav item, not a modal.

**Worked example — "Add Customers to Portal" (Figma source):** Belongs to an existing Portal
detail page, is a multi-step selection task with its own footer summary → **Rank 6, task drawer**,
triggered from a button on the Portal Configuration page, exactly as the Figma frame shows it
layered over `Customer Portal Configurations`.

## 3. Drawer Component

Radix Dialog, anchored to the inline-end edge instead of centered. Two documented compositions —
`DetailDrawer` (read a record) and `TaskDrawer` (do a multi-step/multi-select task) — plus shared
primitives (`DrawerHeader`/`DrawerBody`/`DrawerFooter`).

```
Sizes  : sm 400px · md 480px (DetailDrawer) · lg 620px (TaskDrawer) · xl 760px (dense data)
Anchor : side="end" by default (side="start" reserved for the nav drawer, layout.md §4)
Chrome : full-height · own scroll region in the body · sticky footer · scrim bg-black/40
Mobile : <md collapses to a full-width BOTTOM SHEET (see below) — never a fixed-width side
         panel on a phone
```

```tsx
// components/ui/drawer.tsx
"use client";
import * as Dialog from "@radix-ui/react-dialog";
import { XMarkIcon } from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

const DRAWER_SIZES = { sm: "sm:max-w-[400px]", md: "sm:max-w-[480px]", lg: "sm:max-w-[620px]", xl: "sm:max-w-[760px]" };

export function Drawer({ open, onOpenChange, size = "md", children, closeOnOverlay = true }: {
  open: boolean;
  onOpenChange: (v: boolean) => void;
  size?: keyof typeof DRAWER_SIZES;
  children: React.ReactNode;
  closeOnOverlay?: boolean;   // false while the body has unsaved edits (§4)
}) {
  return (
    <Dialog.Root open={open} onOpenChange={onOpenChange}>
      <Dialog.Portal>
        <Dialog.Overlay
          onClick={e => !closeOnOverlay && e.preventDefault()}
          className={cn(
            "fixed inset-0 z-50 bg-black/40",
            "data-[state=open]:animate-in data-[state=open]:fade-in data-[state=closed]:animate-out data-[state=closed]:fade-out",
            "motion-reduce:animate-none"
          )}
        />
        <Dialog.Content
          onEscapeKeyDown={e => !closeOnOverlay && e.preventDefault()}
          className={cn(
            // Anchored to the inline-end edge; logical properties so RTL flips it automatically.
            "fixed inset-y-0 end-0 z-50 w-full bg-white shadow-lg outline-none flex flex-col",
            "data-[state=open]:animate-in data-[state=open]:slide-in-from-end data-[state=closed]:animate-out data-[state=closed]:slide-out-to-end",
            "duration-200 data-[state=closed]:duration-150 motion-reduce:duration-0 motion-reduce:animate-fade",
            DRAWER_SIZES[size]
          )}
        >
          {children}
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}

export function DrawerHeader({ title, description, onClose }: {
  title: string; description?: string; onClose: () => void;
}) {
  return (
    <div className="shrink-0 flex items-start justify-between gap-4 px-6 py-4 border-b border-border-subtle">
      <div className="min-w-0">
        <Dialog.Title className="font-body text-base font-bold text-foreground truncate">{title}</Dialog.Title>
        {description && (
          <Dialog.Description className="mt-1 font-body text-xs text-muted-foreground">{description}</Dialog.Description>
        )}
      </div>
      <Dialog.Close asChild>
        <button aria-label="Close" onClick={onClose}
          className="shrink-0 w-8 h-8 flex items-center justify-center rounded text-foreground hover:bg-muted">
          <XMarkIcon className="w-6 h-6" />
        </button>
      </Dialog.Close>
    </div>
  );
}

export function DrawerBody({ children, className }: { children: React.ReactNode; className?: string }) {
  return <div className={cn("flex-1 min-h-0 overflow-y-auto px-6 py-6 flex flex-col gap-6", className)}>{children}</div>;
}

export function DrawerFooter({ children }: { children: React.ReactNode }) {
  return (
    <div className="shrink-0 border-t border-border-subtle px-6 py-4 flex flex-col gap-3">
      {children}
    </div>
  );
}
```

### 3a. `DetailDrawer` (Figma `1290:107262`, 480px — "Transaction Details")

Read-only record view: hero amount → 2-col label/value ID card → titled sections (divider under
each title) → sticky footer with a secondary + a destructive action.

```tsx
<Drawer open={open} onOpenChange={setOpen} size="md">
  <DrawerHeader title="Transaction Details" onClose={() => setOpen(false)} />
  <DrawerBody>
    {/* Hero amount */}
    <p className="text-center font-display text-h3 font-bold text-success">875.50 EGP</p>

    {/* ID card: 2-col label/value grid */}
    <div className="border border-border-subtle rounded p-4 grid grid-cols-2 gap-x-6 gap-y-3.5">
      <DetailField label="Status"><StatusChip tone="positive">Success</StatusChip></DetailField>
      <DetailField label="Transaction Type" value="Payment" />
      <DetailField label="Payment Method" value="Portal Wallet" />
      <DetailField label="Reference ID" value="REF-A7B8C9D0" />
      <DetailField label="Device ID" value="POS-T200-014" />
      <DetailField label="Date & Time" value="Yesterday - 01:30 PM" />
    </div>

    {/* Titled section: title → divider → 2-col grid, 32px between sections */}
    <DetailSection title="Cashless Details">
      <DetailField label="Portal" value="Amelia Beirut" />
      <DetailField label="Booth" value="Main Stage - Booth A3" />
      <DetailField label="Card / Wearable" value="NFC Wristband" />
      <DetailField label="Wallet Balance After" value="2,124.50 EGP" />
    </DetailSection>

    <DetailSection title="Customer Information">
      <DetailField label="Customer Name" value="Sara Mohamed" />
      <DetailField label="Customer ID" value="CST-00125" />
      <DetailField label="Phone Number" value="+20 101 345 6789" />
      <DetailField label="Email Address" value="sara.m@email.com" />
    </DetailSection>
  </DrawerBody>
  <DrawerFooter>
    <div className="flex gap-3">
      <Button variant="outline" className="flex-1"><EllipsisHorizontalIcon className="w-[18px] h-[18px]" />More details</Button>
      <Button variant="destructive" className="flex-1"><CreditCardIcon className="w-[18px] h-[18px]" />Refund Transaction</Button>
    </div>
  </DrawerFooter>
</Drawer>

// Shared field/section primitives (reused by any DetailDrawer)
function DetailField({ label, value, children }: { label: string; value?: React.ReactNode; children?: React.ReactNode }) {
  return (
    <div className="flex flex-col gap-1 min-w-0">
      <span className="font-body text-xxs text-cadet-base">{label}</span>
      <span className="font-body text-xs font-semibold text-foreground truncate">{children ?? value}</span>
    </div>
  );
}
function DetailSection({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="flex flex-col gap-4">
      <div className="flex flex-col gap-2">
        <h3 className="font-body text-base font-bold text-foreground">{title}</h3>
        <div className="border-t border-border-subtle" />
      </div>
      <div className="grid grid-cols-2 gap-x-6 gap-y-3.5">{children}</div>
    </div>
  );
}
```

### 3b. `TaskDrawer` (Figma `77:1403`, 620px — "Add Customers to Portal")

A multi-step selection/creation task: context chip → tabs → search → filter chips → scrollable
selectable list (reusing `PickerRow`, `lists.md` §2b) → sticky summary + action footer.

```tsx
<Drawer open={open} onOpenChange={setOpen} size="lg" closeOnOverlay={!hasUnsavedEdits}>
  <DrawerHeader
    title="Add Customers to Portal"
    description="Choose from your existing Customer Module, or create a brand-new customer and enroll them directly into this portal."
    onClose={() => setOpen(false)}
  />
  <DrawerBody className="gap-4">
    {/* Context chip — scopes the task to the record it was launched from */}
    <span className="inline-flex self-start items-center gap-1.5 rounded-full bg-ocean-10 px-3 py-1.5 font-body text-xs font-semibold text-primary">
      <GlobeAltIcon className="w-3.5 h-3.5" /> my-school.kashier.io — This Portal Only
    </span>

    {/* Two equal-width tabs with icon + count */}
    <div className="grid grid-cols-2 border-b border-border">
      <TaskTab active icon={UsersIcon} label="From Existing Customer" count={1420} />
      <TaskTab icon={UserPlusIcon} label="Create New Customer" />
    </div>

    {/* Search — soft gray field, same treatment as lists.md §2 */}
    <div className="flex items-center gap-2 rounded bg-[#F8F8F8] border border-[#F3F3F3] px-4 py-2.5">
      <MagnifyingGlassIcon className="w-4 h-4 text-cadet-05 shrink-0" />
      <input placeholder="Search by name, mobile, email or customer ID…"
        className="flex-1 min-w-0 bg-transparent border-0 font-body text-xs placeholder:text-cadet-05 focus:outline-none" />
    </div>

    {/* Filter chips row (reuse FilterChip, tables.md) */}
    <div className="flex items-center gap-2 flex-wrap">
      <span className="font-body text-xxs font-bold uppercase tracking-wide text-cadet-04">Labels:</span>
      {labels.map(l => <FilterPill key={l.id} selected={l.id === activeLabel}>{l.name}</FilterPill>)}
      <FilterPill>+3 more</FilterPill>
    </div>

    {/* Select-all + count */}
    <div className="flex items-center justify-between">
      <label className="flex items-center gap-2 font-body text-xs font-semibold text-foreground">
        <input type="checkbox" className="w-[15px] h-[15px] rounded border-border" /> Select All (1,420)
      </label>
      <span className="font-body text-xxs text-muted-foreground">Showing all 1,420 customers</span>
    </div>

    {/* Scrollable, groupable picker list */}
    <div className="flex flex-col">
      <p className="font-body text-xxs font-bold uppercase tracking-wide text-cadet-04 py-2">All Customers</p>
      {customers.map(c => <PickerRow key={c.id} {...c} />)}   {/* lists.md §2b */}
    </div>
  </DrawerBody>
  <DrawerFooter>
    <div className="rounded bg-muted px-4 py-3 flex items-center justify-between">
      <span className="font-body text-xs text-foreground">Adding <strong>4 customers</strong> to this portal</span>
      <span className="font-body text-xs text-muted-foreground">Each starts at <strong className="text-foreground">EGP 0.00</strong></span>
    </div>
    <p className="flex gap-2 font-body text-xxs text-muted-foreground">
      <InformationCircleIcon className="w-3.5 h-3.5 shrink-0 mt-0.5" />
      Balances are isolated to this portal. They do not carry over to other portals or the customer's account elsewhere.
    </p>
    <div className="flex gap-3">
      <Button variant="outline" className="flex-1" onClick={() => setOpen(false)}>Cancel</Button>
      <Button className="flex-1">Add to Portal <span className="ms-1 rounded-full bg-white/20 px-1.5">4</span></Button>
    </div>
  </DrawerFooter>
</Drawer>
```

### 3c. Mobile fallback — bottom sheet `<md`

A 620px (or 480px) side panel does not fit a 360px phone. Below `md`, the same `Drawer` renders as
a full-width sheet from the bottom instead of the end edge — swap the anchor classes, don't ship a
second component:

```tsx
// Inside Drawer's Dialog.Content — pick the anchor by breakpoint instead of hand-rolling a 2nd drawer
className={cn(
  "fixed z-50 bg-white shadow-lg outline-none flex flex-col",
  // <md: bottom sheet, rounded top, capped height, drag handle
  "inset-x-0 bottom-0 rounded-t-lg max-h-[90vh]",
  // ≥md: side panel, full height, sized per `size`
  "md:inset-y-0 md:end-0 md:inset-x-auto md:bottom-auto md:rounded-none md:max-h-none md:h-full",
  DRAWER_SIZES[size]
)}
```

```tsx
{/* Drag handle — mobile only */}
<div className="md:hidden mx-auto mt-2 h-1 w-9 rounded-full bg-border shrink-0" />
```

## 4. Motion, Focus & Accessibility

- **Motion**: slide in from the anchored edge, 200ms ease-out on open / 150ms ease-in on close;
  scrim fades independently. Under `prefers-reduced-motion`, drop the slide and cross-fade only
  (`motion-reduce:` variants above) — never disable the transition entirely, a hard cut reads as a
  glitch.
- **Focus**: on open, focus moves to the drawer title (Radix does this by default via
  `Dialog.Content`); on close, focus returns to the trigger element that opened it. Never leave
  focus on `<body>`.
- **Keyboard**: `Esc` closes the drawer — UNLESS it has unsaved edits, in which case `Esc` and
  overlay-click are suppressed (`closeOnOverlay={false}` above) and the close button routes through
  a confirm (§5).
- **Scroll lock**: the page behind the drawer must not scroll while it's open (Radix `Dialog`
  handles this automatically — don't fight it by adding your own body scroll listeners).
- **Labelling**: `Dialog.Title`/`Dialog.Description` (built into `DrawerHeader`) satisfy
  `aria-labelledby`/`aria-describedby` — don't re-implement with a bare `<h2>`.
- **Overlay click**: closes read-only drawers (`DetailDrawer`) by default; for drawers mid-edit
  (`TaskDrawer` with a dirty form) require an explicit Cancel or confirm the discard first.

## 5. Destructive Actions

- Destructive controls sit at the **far end of the footer**, never adjacent to the primary/neutral
  path (Fitts's law — distance is the safety margin).
- Use **filled `destructive`** only when the destructive action IS the drawer's whole purpose (e.g.
  "Refund Transaction" inside the transaction's own detail drawer). Everywhere else, prefer
  `outline`/ghost destructive so it doesn't visually compete with the primary action.
- **Irreversible actions always escalate to a modal confirm** — this is the ladder's one sanctioned
  case of a modal outranking a drawer (§1). A drawer can launch that confirm modal on top of itself;
  don't try to build the confirmation into the drawer's own footer.

## 6. Pre-Build Decision Checklist

Answer these, in order, before writing markup for any new feature or screen:

1. **Surface** — walk the disclosure ladder (§1): inline, drawer, modal, or full page?
2. **Placement** — if this is a new feature (not just new content on an existing page), run the
   placement table (§2). State the chosen rank and why.
3. **Template** — once the surface is chosen, pick the page/drawer template (`layout.md` §7, or
   §3a/§3b above).
4. **Data states** — loaded, loading, empty, and (for a drawer/list) error — all four, not just the
   happy path (Layout Principle 5).
5. **Breakpoint behavior** — what happens `<md`? (bottom sheet for drawers, §3c; list-collapse for
   tables, `tables.md` §5).
6. **One primary action** — does this screen/drawer have exactly one emphasized action? (Visual
   hierarchy law, `SKILL.md`).

Skipping straight to component code without this pass is how modals-for-everything and
nav-item-sprawl happen.
