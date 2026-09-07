# Kashier Components — Code Reference

> shadcn/ui architecture: CSS variables + Tailwind + Radix primitives + `cva()` variants.
> Load this file when implementing or reviewing a specific component.
> Tables & pagination live in `tables.md`. Lists live in `lists.md`. Shell/layout components live in `layout.md`.

## 1. Button

```tsx
// components/ui/button.tsx (shadcn-compatible)
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 font-body text-xs font-semibold " +
  "rounded transition-colors focus-visible:outline-none focus-visible:ring-2 " +
  "focus-visible:ring-ring focus-visible:ring-offset-2 " +
  "disabled:opacity-45 disabled:pointer-events-none",
  {
    variants: {
      variant: {
        default:     "bg-primary text-primary-foreground hover:bg-ocean-90",
        secondary:   "bg-ocean-10 text-primary hover:bg-ocean-20",
        outline:     "border-[1.5px] border-primary text-primary bg-transparent hover:bg-ocean-10",
        ghost:       "text-primary bg-transparent hover:bg-ocean-10",
        accent:      "bg-accent text-accent-foreground hover:bg-kashier-90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-watermelon-90",
        warning:     "bg-warning text-warning-foreground hover:bg-mango-60",
        success:     "bg-success text-success-foreground hover:bg-lemon-80",
        link:        "text-primary underline p-0 h-auto",
      },
      size: {
        sm:      "h-8 px-3 text-[12px]",
        default: "h-10 px-[18px]",
        lg:      "h-12 px-6 text-base",
        xl:      "h-14 px-8 text-base",   /* POS / touch-first primary actions */
        icon:    "h-10 w-10 p-0",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button className={buttonVariants({ variant, size, className })} {...props} />
  );
}
```

**Usage:**
```tsx
<Button>Pay Now</Button>
<Button variant="outline" size="sm">Cancel</Button>
<Button variant="destructive">Delete</Button>
<Button variant="accent"><PlusIcon className="w-4 h-4" />Add Card</Button>
```

## 2. Input

All states are **prop- and class-driven only**. Never rely on the browser's `:invalid` pseudo-class — it triggers on blur regardless of user intent and can't be styled consistently. Always use `noValidate` on the `<form>` element and control error state via the `error` prop.

**State matrix:**

| State | Border | Ring | Background | Cursor |
|-------|--------|------|------------|--------|
| Default | `border-input` (#E6E8E8) | — | white | text |
| Hover | `border-ring/40` | — | white | text |
| Focus | `border-ring` | `ring-ring/12` 3px | white | text |
| Error | `border-destructive` | `ring-destructive/12` 3px on focus | white | text |
| Disabled | `border-input` | — | `bg-muted` + opacity-60 | not-allowed |
| Read-only | `border-input` | — | `bg-muted` | default |

```tsx
// components/ui/input.tsx
import { cn } from "@/lib/utils";

export interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  /** Triggers error border + ring. Always pair with a visible errorMessage in FormField. */
  error?: boolean;
}

export function Input({ className, error, ...props }: InputProps) {
  return (
    <input
      aria-invalid={error || undefined}
      className={cn(
        // Base
        "flex h-10 w-full rounded border-[1.5px] border-input bg-background",
        "px-3 font-body text-xs text-foreground placeholder:text-cadet-05",
        "transition-colors",
        // Hover
        "hover:border-ring/40",
        // Focus
        "focus-visible:outline-none focus-visible:border-ring",
        "focus-visible:ring-[3px] focus-visible:ring-ring/12",
        // Disabled
        "disabled:bg-muted disabled:opacity-60 disabled:cursor-not-allowed disabled:pointer-events-none",
        // Read-only
        "read-only:bg-muted read-only:cursor-default",
        // Error — prop-driven, overrides hover/focus border+ring
        error && [
          "border-destructive hover:border-destructive",
          "focus-visible:border-destructive focus-visible:ring-destructive/12",
        ],
        // Suppress browser-native validation shadow (box-shadow injected by :invalid)
        "[&:invalid]:shadow-none",
        className
      )}
      {...props}
    />
  );
}
```

**`FormField` — label + hint + error message:**
```tsx
// components/ui/form-field.tsx
import { cn } from "@/lib/utils";

export function FormField({
  label, hint, error, errorMessage, required, children, className,
}: {
  label?: string;
  hint?: string;
  error?: boolean;
  errorMessage?: string;
  required?: boolean;
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <div className={cn("flex flex-col gap-1.5", className)}>
      {label && (
        <label className="font-body text-xs font-semibold text-foreground">
          {label}
          {required && <span className="text-destructive ms-0.5" aria-hidden>*</span>}
        </label>
      )}
      {children}
      {error && errorMessage ? (
        <p className="font-body text-xxs text-destructive" role="alert">{errorMessage}</p>
      ) : hint ? (
        <p className="font-body text-xxs text-muted-foreground">{hint}</p>
      ) : null}
    </div>
  );
}
```

**Usage — all states:**
```tsx
{/* Default with hint */}
<FormField label="Card Holder Name" hint="Enter the name as it appears on the card.">
  <Input placeholder="John Doe" />
</FormField>

{/* Error — prop-driven, not browser validation */}
<FormField label="Email" error errorMessage="Invalid email address." required>
  <Input type="email" placeholder="you@example.com" error />
</FormField>

{/* Disabled */}
<FormField label="Merchant ID"><Input value="KSH-00123" disabled /></FormField>

{/* Read-only */}
<FormField label="Reference"><Input value="TXN-98765" readOnly /></FormField>

{/* Always add noValidate — opt out of browser validation entirely */}
<form noValidate onSubmit={handleSubmit}>
  <FormField label="Amount" error={!!errors.amount} errorMessage={errors.amount} required>
    <Input type="number" placeholder="0.00" error={!!errors.amount} />
  </FormField>
</form>
```

## 3. Badge

```tsx
// components/ui/badge.tsx
import { cva, type VariantProps } from "class-variance-authority";

const badgeVariants = cva(
  "inline-flex items-center gap-1 rounded-full font-body text-[11px] font-bold " +
  "uppercase tracking-[0.3px] px-2.5 py-0.5",
  {
    variants: {
      variant: {
        primary:     "bg-ocean-10 text-ocean-80",
        accent:      "bg-kashier-10 text-kashier-100",
        success:     "bg-lemon-10 text-lemon-80",
        warning:     "bg-mango-10 text-mango-80",
        error:       "bg-watermelon-10 text-watermelon-base",
        neutral:     "bg-cadet-09 text-cadet-02",
        secondary:   "bg-[#FFF5F0] text-[#8A4928]",
      },
    },
    defaultVariants: { variant: "primary" },
  }
);

export function Badge({ variant, className, ...props }:
  React.HTMLAttributes<HTMLSpanElement> & VariantProps<typeof badgeVariants>) {
  return <span className={badgeVariants({ variant, className })} {...props} />;
}
```

**Transaction status usage:**
```tsx
const statusVariant = {
  paid:       "success",
  pending:    "warning",
  failed:     "error",
  refunded:   "neutral",
  processing: "primary",
} as const;

<Badge variant={statusVariant[transaction.status]}>
  <span className="w-1.5 h-1.5 rounded-full bg-current" />
  {transaction.status}
</Badge>
```

> Status in **tables, record cards, and POS** uses `StatusChip` (radius 2, Noto Sans regular — `tables.md`), and user-assigned labels use `LabelChip` (Neutral 09 pill — `lists.md`). The pill Badge above is only for page-level statuses where a bolder treatment is wanted.

## 4. Alert

Figma node `96:1292` — exact color spec in `figma-exact.md`. Props: `type` × `withTitle` × `rtl`.

Key rules: full 1px border (not left-only), `border-radius: 5px` (not the global 4px), `padding: 12px`, `gap: 8px`, 24px Heroicons outline icon, icon left in LTR / right in RTL, title and message share the same color.

```tsx
// components/ui/alert.tsx
import {
  CheckCircleIcon,
  ExclamationCircleIcon,
  ExclamationTriangleIcon,
  InformationCircleIcon,
} from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

const alertConfig = {
  error:   { bg: "bg-[#FFEAED]", border: "border-[#FFBBC4]", text: "text-[#A50017]", Icon: ExclamationCircleIcon },
  success: { bg: "bg-[#F3FFF4]", border: "border-[#AEFFB2]", text: "text-[#1A541D]", Icon: CheckCircleIcon },
  warning: { bg: "bg-[#FFF7E8]", border: "border-[#FFD484]", text: "text-[#513500]", Icon: ExclamationTriangleIcon },
  info:    { bg: "bg-[#F1F6FF]", border: "border-[#C0D5FF]", text: "text-[#001F5F]", Icon: InformationCircleIcon },
} as const;

export interface AlertProps {
  type?: keyof typeof alertConfig;
  title?: string;
  message: string;
  rtl?: boolean;
  className?: string;
}

export function Alert({ type = "info", title, message, rtl = false, className }: AlertProps) {
  const { bg, border, text, Icon } = alertConfig[type];
  return (
    <div
      className={cn(
        "flex gap-2 p-3 border rounded-[5px]",   // ← 5px, NOT rounded-md
        rtl && "flex-row-reverse text-end",
        bg, border, text,
        rtl ? "font-body-ar" : "font-body",
        className
      )}
      dir={rtl ? "rtl" : "ltr"}
    >
      <Icon className="w-6 h-6 shrink-0" />
      <div className={cn("flex flex-col gap-[5px] pt-1", rtl && "items-end")}>
        {title && <p className="text-[14px] font-bold leading-[1.2]">{title}</p>}
        <p className="text-[14px] font-normal leading-[1.2]">{message}</p>
      </div>
    </div>
  );
}
```

**Usage:**
```tsx
<Alert type="error"   message="Compact error message!" />
<Alert type="success" message="Payment processed successfully." />
<Alert type="error" title="Payment Failed" message="Insufficient funds. Please try another card." />
<Alert type="success" message="تمت معالجة الدفعة بنجاح." rtl />
```

## 5. Card

Figma node `174:8655` (file `lVNlJ8QUx3McAwnkJlXhHS`) — exact spec table in `figma-exact.md`.

> ⚠️ **Card border = `border-subtle` NOT `border`.**
> `--border` (`#E6E8E8`) is for inputs and table rows.
> `--border-subtle` (`#F5F6F6`) is for cards, header separators, and internal dividers.

**Two card variants:**
- `variant="flat"` (default) — border only, no shadow → data/summary cards
- `variant="raised"` — border + `shadow-md` → widget/stat cards

```tsx
// components/ui/card.tsx
import { cn } from "@/lib/utils";

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: "flat" | "raised";
}

export function Card({ className, variant = "flat", ...props }: CardProps) {
  return (
    <div
      className={cn(
        "bg-white border border-border-subtle rounded overflow-hidden",
        variant === "raised" && "shadow-md",
        className
      )}
      {...props}
    />
  );
}

// Inner wrapper: p-4 (16px) with gap-6 (24px) between sections
export function CardInner({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn("p-4 flex flex-col gap-6", className)} {...props} />;
}

// Header: icon + Outfit Bold H5 title, bottom border border-subtle
export function CardHeader({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div className={cn("flex items-center gap-2 pb-2 border-b border-border-subtle", className)} {...props} />
  );
}

// Header icon: 24px, color cadet-base (#556767)
export function CardHeaderIcon({ icon: Icon }: { icon: React.ComponentType<{ className?: string }> }) {
  return <Icon className="w-6 h-6 text-cadet-base shrink-0" />;
}

// Header title: Outfit Bold 25px, color cadet-base (#556767)
// NOTE: table cards and chart/widget cards in the portal use a smaller 18px
// semibold cadet-base title (see TableToolbar in tables.md) — the 25px CardTitle
// is for summary/detail cards.
export function CardTitle({ className, ...props }: React.HTMLAttributes<HTMLHeadingElement>) {
  return (
    <h3 className={cn("font-display text-h5 font-bold leading-[1.2] text-cadet-base", className)} {...props} />
  );
}

// Named section within a card
export function CardSection({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn("flex flex-col gap-2", className)} {...props} />;
}

// Section label: Noto Sans Bold 16px, cadet-base
export function CardSectionLabel({ className, ...props }: React.HTMLAttributes<HTMLParagraphElement>) {
  return (
    <p className={cn("font-body text-base font-bold leading-[1.2] text-cadet-base", className)} {...props} />
  );
}

// Data row: label ↔ value, justify-between
export function CardRow({ label, value, valueClassName }: {
  label: string;
  value: string;
  valueClassName?: string;
}) {
  return (
    <div className="flex items-center justify-between gap-4 py-1.5">
      <span className="font-body text-xs font-normal leading-[1.2] text-cadet-base">{label}</span>
      <span className={cn("font-body text-base font-semibold leading-[1.2] text-foreground text-end", valueClassName)}>
        {value}
      </span>
    </div>
  );
}

// Subtle divider: same color as card border
export function CardDivider({ className }: { className?: string }) {
  return <div className={cn("h-px w-full bg-border-subtle", className)} />;
}

// Footer: action area, muted bg, border-subtle top
export function CardFooter({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={cn("px-4 py-3 bg-muted border-t border-border-subtle flex justify-end gap-2.5", className)}
      {...props}
    />
  );
}
```

**Usage:**
```tsx
<Card>
  <CardInner>
    <CardHeader>
      <CardHeaderIcon icon={BookmarkIcon} />
      <CardTitle>Summary</CardTitle>
    </CardHeader>

    <CardSection>
      <CardSectionLabel>Activity</CardSectionLabel>
      <CardRow label="Gross amount" value="10,243.59 EGP" />
      <CardRow label="Fees"         value="100.59 EGP" />
      <CardDivider />
      <CardRow label="Net amount"   value="10,143.59 EGP" />
    </CardSection>
  </CardInner>
</Card>

{/* Raised variant with footer */}
<Card variant="raised">
  <CardInner>
    <CardHeader>
      <CardHeaderIcon icon={ShieldCheckIcon} />
      <CardTitle>KYC Status</CardTitle>
    </CardHeader>
    <p className="font-body text-xs text-muted-foreground">Description text here.</p>
  </CardInner>
  <CardFooter>
    <Button variant="ghost" size="sm">Cancel</Button>
    <Button size="sm">Confirm</Button>
  </CardFooter>
</Card>
```

## 6. Avatar

```tsx
// components/ui/avatar.tsx
const avatarVariants = cva(
  "rounded-full flex items-center justify-center font-display font-bold " +
  "text-white shrink-0 tracking-wide",
  {
    variants: {
      size: {
        xs: "w-6 h-6 text-[9px]",
        sm: "w-8 h-8 text-[11px]",
        md: "w-10 h-10 text-sm",
        lg: "w-14 h-14 text-lg",
        xl: "w-18 h-18 text-2xl",
      },
      color: {
        primary:   "bg-primary",
        accent:    "bg-accent",
        secondary: "bg-secondary text-secondary-foreground",
        neutral:   "bg-cadet-04",
        success:   "bg-success",
      },
    },
    defaultVariants: { size: "md", color: "primary" },
  }
);

export function Avatar({ initials, src, size, color, className }: {
  initials?: string;
  src?: string;
  size?: "xs"|"sm"|"md"|"lg"|"xl";
  color?: "primary"|"accent"|"secondary"|"neutral"|"success";
  className?: string;
}) {
  if (src) {
    return <img src={src} className={avatarVariants({ size, color, className })} alt="" />;
  }
  return <div className={avatarVariants({ size, color, className })}>{initials}</div>;
}
```

## 7. Select

```tsx
// Use shadcn Select (Radix) with Kashier tokens
import * as SelectPrimitive from "@radix-ui/react-select";

// Trigger
<SelectPrimitive.Trigger className="flex h-10 w-full items-center justify-between rounded border-[1.5px] border-input bg-background px-3 font-body text-xs text-foreground focus:outline-none focus:border-ring focus:ring-[3px] focus:ring-ring/12 disabled:opacity-60">
  <SelectPrimitive.Value placeholder="Select currency..." />
  <ChevronDownIcon className="w-4 h-4 text-muted-foreground" />
</SelectPrimitive.Trigger>

// Content
<SelectPrimitive.Content className="z-50 min-w-[8rem] rounded border border-border bg-popover shadow-lg">
  <SelectPrimitive.Item className="relative flex cursor-pointer select-none items-center px-3 py-2.5 text-xs text-foreground outline-none hover:bg-muted focus:bg-muted">
    ...
  </SelectPrimitive.Item>
</SelectPrimitive.Content>
```

## 8. Checkbox

Figma spec (component `5:445`/`5:407`): 16px, radius 4, border `#D5D9D9` (Neutral 07/`cadet-07`). Checked = **navy outline + navy check — NOT a filled box**.

```tsx
// Use shadcn Checkbox (Radix) with Kashier tokens
import * as CheckboxPrimitive from "@radix-ui/react-checkbox";

<CheckboxPrimitive.Root
  className="w-4 h-4 rounded border border-cadet-07 bg-background
    data-[state=checked]:border-primary
    focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-1
    transition-colors"
>
  <CheckboxPrimitive.Indicator>
    <CheckIcon className="w-3 h-3 text-primary" strokeWidth={3} />
  </CheckboxPrimitive.Indicator>
</CheckboxPrimitive.Root>

// Optional label: Noto Sans Regular 12px, 8px gap
```

## 9. Switch

```tsx
// Use shadcn Switch (Radix) with Kashier tokens
import * as SwitchPrimitive from "@radix-ui/react-switch";

<SwitchPrimitive.Root
  className="relative inline-flex h-6 w-[42px] shrink-0 cursor-pointer rounded-full border-2 border-transparent
    bg-border transition-colors
    data-[state=checked]:bg-primary
    focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2"
>
  <SwitchPrimitive.Thumb
    className="pointer-events-none block h-5 w-5 rounded-full bg-white shadow-sm
      transition-transform data-[state=checked]:translate-x-[18px] data-[state=unchecked]:translate-x-0"
  />
</SwitchPrimitive.Root>
```

## 9b. SwitchEnhanced + ModeChip (test/live mode — Figma `2018:43798` / `3496:24650`)

The topbar mode toggle. Chip: `p-2 rounded gap-4`, label Noto Sans Regular 16px. Switch 33×20 with bordered track + bordered knob and an inset track shadow — richer than the plain Switch above.

| Mode | Chip bg / border | Track / border | Knob / border |
|------|------------------|----------------|---------------|
| Test | `#FFF7E8` / `#FFE6B6` (Warning 09/08) | `#FBAF1F` / `#D9930E` | `#FFF7E8` / `#B77801` |
| Live | `#F3FFF4` / `#4AAB4E` (Success 09/Base) | `#4AAB4E` / `#388E3B` | `#F3FFF4` / `#28712B` |

Geometry (from the Figma component): the control is 33×20 overall; the **track is 16px tall** (inset 2px top/bottom, full width, pill radius, inset shadow) and the **knob is a full-height 20px circle that overhangs the track** — knob at the start in test mode, at the end in live mode.

```tsx
export function ModeChip({ mode, onChange }: { mode: "test" | "live"; onChange: (m: "test" | "live") => void }) {
  const test = mode === "test";
  return (
    <button
      onClick={() => onChange(test ? "live" : "test")}
      role="switch"
      aria-checked={!test}
      className={cn(
        "flex items-center gap-4 p-2 rounded border font-body text-base text-warning-foreground",
        test ? "bg-mango-10 border-[#FFE6B6]" : "bg-lemon-10 border-success",
      )}
    >
      {test ? "Test mode" : "Live mode"}
      <span className="relative inline-block w-[33px] h-5 shrink-0">
        {/* Track: 16px tall, full width, inset shadow */}
        <span className={cn(
          "absolute inset-x-0 inset-y-0.5 rounded-full border shadow-[inset_0_0_4px_rgba(0,0,0,0.4)] transition-colors",
          test ? "bg-warning border-mango-60" : "bg-success border-lemon-80",
        )} />
        {/* Knob: full-height 20px circle overhanging the track */}
        <span className={cn(
          "absolute top-0 h-5 w-5 rounded-full border shadow-[0_0_2px_rgba(0,0,0,0.25)] transition-all",
          test ? "start-0 bg-mango-10 border-mango-70" : "start-[13px] bg-lemon-10 border-lemon-90",
        )} />
      </span>
    </button>
  );
}
```

## 9c. Modal / Dialog (Figma Balances `1254:24866`)

> **Modal is the second choice, not the default overlay.** Run the disclosure ladder in
> `patterns.md` §1 first — a **`Drawer`** (`patterns.md` §3) is the default for record detail,
> multi-field/multi-step tasks, and list browsing/selection. Reach for this centered Modal only for
> the three cases the ladder reserves for it: a destructive confirmation, a genuinely short
> (≤3-field) single-purpose form, or a blocking system message. The payout-setup example below is
> a legitimate case — it's a single-purpose, ≤3-field form with nothing else on the page to keep
> visible behind it.

Radix Dialog. Centered card ~576px over a dimmed (optionally blurred) backdrop. Anatomy: illustration + Bold title + gray description → optional **segmented choice row** → form fields on a light muted panel → footer with end-aligned actions.

```tsx
import * as Dialog from "@radix-ui/react-dialog";

<Dialog.Portal>
  <Dialog.Overlay className="fixed inset-0 z-50 bg-black/30 backdrop-blur-[2px]" />
  <Dialog.Content className="fixed z-50 top-1/2 start-1/2 -translate-x-1/2 -translate-y-1/2 w-[min(576px,calc(100vw-32px))] bg-white rounded shadow-lg p-6 flex flex-col gap-5 focus:outline-none">
    {/* Header: illustration + title + description */}
    <div className="flex gap-4 items-center">
      <img src={illustration} className="w-16 h-16 shrink-0" alt="" />
      <div>
        <Dialog.Title className="font-body text-base font-bold text-foreground">Let's setup your payouts</Dialog.Title>
        <Dialog.Description className="font-body text-xs text-muted-foreground">
          Please add your primary account payout details to receive payments.
        </Dialog.Description>
      </div>
    </div>

    {/* Segmented choice row — selected = primary border + primary text */}
    <div className="flex gap-3">
      {options.map(o => (
        <button key={o.id} onClick={() => setChoice(o.id)}
          className={cn(
            "flex-1 flex items-center justify-center gap-2 h-10 rounded border font-body text-xs font-semibold",
            choice === o.id ? "border-primary text-primary" : "border-border text-cadet-base hover:border-primary/40",
          )}>
          <o.icon className="w-[18px] h-[18px]" />{o.label}
        </button>
      ))}
    </div>

    {/* Form panel on muted bg */}
    <div className="bg-muted/60 rounded p-4 flex flex-col gap-4">
      <FormField label="Recipient Full Name"><Input placeholder="Recipient Full Name" /></FormField>
      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <FormField label="Recipient Bank"><Select /></FormField>
        <FormField label="Recipient Account Number"><Input /></FormField>
      </div>
    </div>

    {/* Footer: end-aligned, secondary outline + small primary */}
    <div className="flex justify-end gap-2.5">
      <Dialog.Close asChild><Button variant="outline" size="sm">I'll do that later</Button></Dialog.Close>
      <Button size="sm">Confirm</Button>
    </div>
  </Dialog.Content>
</Dialog.Portal>
```

## 10. Tabs

```tsx
// Use shadcn Tabs (Radix) with Kashier tokens
import * as TabsPrimitive from "@radix-ui/react-tabs";

<TabsPrimitive.List className="border-b-2 border-border flex gap-0 overflow-x-auto">
  {tabs.map(tab => (
    <TabsPrimitive.Trigger
      key={tab.value}
      value={tab.value}
      className="px-5 py-2.5 font-body text-xs font-semibold text-muted-foreground
        border-b-2 border-transparent -mb-0.5 transition-colors whitespace-nowrap
        data-[state=active]:text-primary data-[state=active]:border-primary
        hover:text-foreground"
    >
      {tab.label}
    </TabsPrimitive.Trigger>
  ))}
</TabsPrimitive.List>
```

## 11. Breadcrumb

```tsx
export function Breadcrumb({ items }: { items: { label: string; href?: string }[] }) {
  return (
    <nav className="flex items-center gap-1.5 flex-wrap font-body text-xs">
      {items.map((item, i) => (
        <React.Fragment key={i}>
          {i > 0 && <span className="text-cadet-05 text-[11px]">/</span>}
          {item.href ? (
            <a href={item.href} className="text-primary hover:underline">{item.label}</a>
          ) : (
            <span className="text-foreground font-semibold">{item.label}</span>
          )}
        </React.Fragment>
      ))}
    </nav>
  );
}
```

## 12. Account Selector

Figma node `183:7867` (Balances file) — exact spec in `figma-exact.md`.

```tsx
// components/ui/AccountSelector.tsx
import { EllipsisHorizontalCircleIcon } from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";

type Account = { id: string; name: string; sub?: string; balance: string };

function AccountCard({ account, selected, onSelect }: {
  account: Account; selected: boolean; onSelect: () => void;
}) {
  return (
    <button
      onClick={onSelect}
      className={cn(
        "flex-1 min-w-[140px] flex flex-col gap-2 p-4 rounded border text-start transition-colors bg-white",
        selected ? "border-primary" : "border-border hover:border-primary/40"
      )}
    >
      <div className="flex flex-col gap-0.5">
        <span className="text-sm font-semibold text-cadet-base leading-snug">{account.name}</span>
        {account.sub && <span className="text-xs text-cadet-04 leading-snug">{account.sub}</span>}
      </div>
      <span className={cn("text-base font-bold leading-snug", selected ? "text-foreground" : "text-cadet-base")}>
        {account.balance}
      </span>
    </button>
  );
}

export function AccountSelector({ accounts, selectedId, onSelect }: {
  accounts: Account[]; selectedId: string; onSelect: (id: string) => void;
}) {
  return (
    <div className="flex gap-4 flex-wrap">
      {accounts.map(acc => (
        <AccountCard key={acc.id} account={acc} selected={acc.id === selectedId} onSelect={() => onSelect(acc.id)} />
      ))}
      <div className="flex-none min-w-[130px] border border-border rounded p-4 flex flex-col gap-2 cursor-pointer hover:border-primary/40 transition-colors">
        <EllipsisHorizontalCircleIcon className="w-5 h-5 text-cadet-base" />
        <span className="text-sm font-semibold text-cadet-base">Other accounts…</span>
      </div>
    </div>
  );
}
```

## 13. KPI Widget Bar

Figma node `10:4785` (Balances file) — exact spec in `figma-exact.md`. Responsive grid recipe in `layout.md`.

**Border rule: ONE outer border on the container, and exactly ONE 1px divider between cells** — never give each cell its own border (adjacent borders double up into a 2px line). The `gap-px` over a `bg-border-subtle` container produces the single dividers in both the desktop row and the 2×2 mobile grid.

**Text spacing inside a cell (Figma `10:4785`, proximity): label + value are ONE unit — zero gap between them; the description sits 8px below the unit.** Never distribute the three lines with uniform gaps.

```tsx
// components/ui/KpiBar.tsx
type KpiItem = { label: string; value: string; description?: string };

export function KpiBar({ items }: { items: KpiItem[] }) {
  return (
    // Container owns the border; cells are separated by 1px of the container's bg.
    <div className="grid grid-cols-2 md:flex gap-px rounded overflow-hidden border border-border-subtle bg-border-subtle">
      {items.map(item => (
        <div key={item.label} className="flex-1 min-w-0 bg-white flex flex-col gap-2 p-4">
          {/* label + value: one unit, no gap between them */}
          <div className="flex flex-col">
            <span className="font-body text-xs font-normal text-cadet-base uppercase leading-[1.2]">
              {item.label}
            </span>
            {/* 25px in half-width mobile cells, 31px ≥md — a fixed 31px overflows phones */}
            <span className="font-display text-h5 md:text-h4 font-bold text-foreground leading-[1.2] [word-break:break-word]">
              {item.value}
            </span>
          </div>
          {item.description && (
            <span className="font-body text-xxs text-cadet-06 leading-[1.2]">{item.description}</span>
          )}
        </div>
      ))}
    </div>
  );
}
```

## 14. Icon Rules

Use **Heroicons** exclusively. Import from `@heroicons/react/24/outline` (default) or `/24/solid`.

The product uses a three-step icon scale (prototype classes `.hi` / `.hi-sm` / `.hi-xs`):

| Size | Use |
|------|-----|
| 18px `w-[18px] h-[18px]` | **Default** — nav links, buttons, toolbar links |
| 16px `w-4 h-4` | Small buttons, inline chips, search magnifier |
| 14px `w-3.5 h-3.5` | Dense contexts: pagination arrows, tiny affordances |
| 24px `w-6 h-6` | Alerts, card headers, standalone feature icons |

**Verbatim Heroicons only — never hand-draw or approximate a path.** Wrong or hand-written path data is how icons end up "not properly inserted":

- **React**: import from `@heroicons/react/24/outline` (or `/24/solid`, `/16/solid`) — the package ships the exact vectors.
- **Plain HTML/prototypes**: copy the **exact SVG markup from heroicons.com** (Copy SVG button) — full path data, `viewBox="0 0 24 24"`, `fill="none" stroke="currentColor"` for outline. Never retype, truncate, or "reconstruct" a `d` attribute from memory.
- Verify the glyph name exists in Heroicons before using it; if a concept has no Heroicon, say so and pick the closest real one — don't invent a shape.
- Size with explicit width/height classes on the `<svg>`; color via `currentColor`.

```tsx
import { PlusIcon, ChevronDownIcon, XMarkIcon } from "@heroicons/react/24/outline";

// Sidebar/nav icons use stroke-width 1.5 (lighter, matches the prototypes);
// everywhere else keep the Heroicons default stroke-width 2 (outline set is drawn at 1.5,
// the prototypes override with stroke-width attribute where needed).
```

## 15. RTL / Arabic Rules

```tsx
// Always use dir="rtl" on Arabic containers, NOT on the whole document
// unless the page is 100% Arabic.

// Pattern for bilingual pages:
<div dir={locale === "ar" ? "rtl" : "ltr"} className={locale === "ar" ? "font-body-ar" : "font-body"}>
  {content}
</div>

// Text alignment: use logical properties
// ✅ text-start (not text-left)
// ✅ ms-4 / me-4 (not ml-4 / mr-4) for margins
// ✅ ps-3 / pe-3 (not pl-3 / pr-3) for padding
// ✅ rounded-s-md / rounded-e-md for directional rounding

// Arabic xSmall font size is 13px (text-xs-ar), NOT 14px
// Arabic xxSmall is 10px (text-xxs-ar), NOT 12px
```
