# Kashier Platforms — Responsive Web, Mobile App, POS Terminal

> Load this file when the target is not a desktop browser: native mobile app, POS terminal,
> or when deciding how a feature adapts across platforms. Same tokens everywhere — what changes
> is chrome, density, and input model.

## 1. Platform Decision Table

| | Responsive web | Native mobile app | POS terminal |
|---|---|---|---|
| Viewport | 320 → ∞, fluid | 360–430 logical px | **720×1440 fixed** (Android smart-POS) |
| Nav pattern | sidebar ≥lg, drawer <lg | 5-slot tab bar + **center FAB** | app bar + 3-slot bottom icon nav |
| Base body size | 14px (`text-xs`) | 14–16px | **25px (Outfit H5)** |
| Touch target min | 44px on touch | 44px | **64px+ (buttons 96px)** |
| Hover states | yes (pointer) | **no** — press feedback | **no** — press feedback |
| Drawer (detail/task, default overlay — `patterns.md` §1/§3) | end-anchored side panel | bottom sheet or full-screen push | fullscreen step |
| Modal (confirm/short-form only) | centered modal | centered modal / bottom sheet | fullscreen step |
| Sheet (action menu) | rarely used (prefer dropdown) | bottom sheet | fullscreen step |
| Data display | tables ≥md, lists <md | lists only | shadowed cards, large single values |
| Density | comfortable | comfortable | **extra large** |

**Touch platforms have no hover.** Every `hover:` needs an `active:` counterpart (`active:bg-…`, `active:opacity-80`) so touch users get press feedback.

**Drawer is the overlay default on every platform** (`patterns.md` §1) — the difference is only the
container shape: an end-anchored panel on web, a bottom sheet (or full-screen push, for content-
heavy tasks) on mobile, a fullscreen step on POS (POS has no room for a partial-width panel at all).

## 2. Responsive Web

Canonical rules live in `layout.md` (breakpoints §3, AppShell §4, templates §7). Summary: mobile-first classes, drawer nav `<lg`, tables→lists `<md`, actions full-width `<sm`.

## 3. Native Mobile App

Patterns are framework-agnostic; examples use React Native `StyleSheet` values (the DS is React-based — map 1:1 to Flutter/native). Reuse the token values from `tokens.md` as constants:

```ts
// theme/tokens.ts (React Native)
export const colors = {
  primary: "#001F5F", accent: "#00BCB4", foreground: "#202020",
  mutedForeground: "#637373", border: "#E6E8E8", borderSubtle: "#F5F6F6",
  muted: "#F5F6F6", destructive: "#A50017", success: "#4AAB4E", warning: "#FBAF1F",
  surface: "#FFFFFF", sidebarBg: "#F8F9F9",
};
export const radius = { sm: 2, md: 4, lg: 8, full: 999 };
export const spacing = { xs: 4, sm: 8, md: 12, lg: 16, xl: 24, xxl: 32 };
// Fonts: Outfit (display) / Noto Sans (body); Cairo / Noto Naskh Arabic for AR
```

**Sources**: Payment Links app file `D38J1PWGdciEbyJq5RrpCS` (sections: List `21:9112`, Creation `22:9533`, Details `225:13476`; record card `911:21641`) · Home-Balance file `gWH3omAqpHlhjqkAGg9zx7` (`461:4667`). Measurement blocks in `figma-exact.md`.

### 3a. Screen anatomy & bottom tab bar

```
SafeAreaView (bg white)
 └─ AppBar     title Outfit Bold 20px start · end: inline 24px action icons (search, filter, plus)
               back chevron replaces title-start on pushed screens (44px hit slop)
 └─ FilterTabs optional — All / Paid / Unpaid / Overdue · compact underline tabs (Tabs pattern)
 └─ Content    ScrollView/FlatList · px 16 · sections gap 24
 └─ TabBar     5 slots with a CENTER FLOATING FAB:
               [Invoices] [Transactions]  (+)  [Notifications] [Settings]
               FAB = navy circle, elevated above the bar, white plus icon
               side items = 24px icon + 10px label · active navy (solid icon) · inactive muted
```

### 3b. Compact record card (`911:21641`, 342×116)

The app's entity list item — a lighter cousin of the portal RecordCard, day-grouped the same way:

```
Card  : bg white · border 1.5px #F8F9F9 (Neutral 10) · radius 4 · p-4 · rows gap-2
Row 1 : StatusChip (10px text variant) + ref 12px #839090  ↔  amount Noto Sans Bold 14px
Row 2 : 18px Heroicon + name SemiBold 14px #556767 · sub ("Due tomorrow") 12px #839090
Row 3 : indent 22px — delivery mini-chips: 0.5px border · radius 2 · px-1 py-px
        14px icon + 10px uppercase text
        Delivered: bg #F3FFF4 · border #D1FFD3 · text #4AAB4E
        Pending  : bg rgba(255,247,232,.48) · border #FFE6B6 · text #B77801
Kebab : end of row 2 → opens the action bottom sheet
```

### 3c. Sheets

> A bottom sheet is also the `<md` fallback for the web `Drawer` (`patterns.md` §3c) — don't design
> a second bespoke sheet shape for that case; reuse this one.

- **Action sheet** (record kebab): title = record name + sub, then icon rows — Details, Edit, Share, Split payment, Mark as paid, Clone, and Delete in destructive red at the bottom.
- **Share sheet**: "Share with QR code" primary-outline button → copy-link field with Copy button → Send Email row (input + Send) → Send SMS row (country code + number + Send).
- Sheets: rounded-t-lg, drag handle, actions as `ListItem` rows (lists.md §3).

### 3d. Home screen anatomy (Home-Balance `461:4667`)

```
Greeting  : "Hello, {Name}!" — name Outfit Bold in primary · date 12px muted · bell w/ badge end
Services  : section header + "All services →" link · 2-col tile grid
            tile = small card: 24px icon, SemiBold 14px title, 12px muted caption
Reports   : section header + "Today ▾" scope chip · "All accounts ▾" account chip
            stat card on muted bg: uppercase 12px label + Outfit Bold ~28px amount
            + secondary stat columns · then compact filter tabs (All balance/Payments/…)
            + compact transaction rows · "Go to balance →" outline button full-width
ComingSoon: feature placeholder — 48px icon in muted circle · SemiBold title
            · orange "COMING SOON" chip · 12px description
```

### 3e. Content rules

- Entity lists = day-grouped compact record cards (§3b) — never tables.
- Search: icon in the app bar expands to a full-width field under the title.
- States are mandatory: illustrated empty state + CTA ("Ready to get paid?" + New payment link) · no-results with "Clear search"/"Clear filters" outline button · skeleton screens while loading.
- Forms: one column, labels above fields, 48px inputs, submit as full-width bottom button.
- Pull-to-refresh; infinite scroll — no numbered pagination.
- Press feedback: opacity 0.7 or `muted` bg; ≥44px targets; 8px between adjacent targets.

## 4. POS Terminal

**Source of truth: the POS Figma file `nhbdiaZtsdroBgJnRbhC77`, "User Flow" page** (key node ids below; measurement blocks also in `figma-exact.md`). Device: Android smart-POS, portrait, **720×1440** logical px — Android status bar (48px) on top, system nav bar (96px) at the bottom; app content lives between them. Content width **664px** (28px side margins).

### 4a. POS design language

- **Type is all Outfit** (Cairo for Arabic) — Noto Sans appears only inside status chips:
  | Role | Style |
  |------|-------|
  | Amounts (hero) | Lead Bold **76px**, `#001F5F` navy, centered |
  | Result titles | H2 Bold **49px** (Success 01 `#1A541D` / Error Base `#A50017`) |
  | App bar title, button labels | H4 Bold **31px** |
  | Body, card text, subtitles | H5 **25px** (Regular/Bold) |
  | Section headers ("Today") | H4 Bold 31px, Neutral 02 `#697979` |
- **Palette naming**: this file calls the navy scale "Accent" (Base #001F5F, 01 #103788, 07 #C0D5FF, 09 #F1F6FF) — it is the Ocean palette. Cards use Neutral 11 `#FDFDFD`; shadows are Level 1 `0 0 5px rgba(0,0,0,.25)`.
- One task per screen; no sidebar/breadcrumbs/tabs; no hover — `active:` press feedback.

### 4b. Building blocks

**App bar** — bg `#F8F9F9`, ~112px content height: 32px back arrow at inline-start 28px, title Outfit Bold 31px `#202020`.

**Buttons**
| Kind | Spec |
|------|------|
| Primary (full-width) | 664×**96px** · radius 8 · bg `#001F5F` · label Outfit Bold 31px white |
| Secondary (full-width) | 664×96 · radius 8 · white bg · 1px `#001F5F` border · 50px icon at start, Outfit Bold 31px navy label at end (justify-between) |
| Pill | radius 44 · px-6 py-4 · 25px label (e.g. "Reports", navy solid) |
| Squared action tile | **322×304** · radius 25 · border-2 `#103788` · shadow `0 0 10px rgba(0,0,0,.25)` · navy circle with 82px white Heroicon · label Outfit 31px navy · 2-col grid |
| Round utility | **64px** circle · bg `#F1F6FF` · 32px Heroicon (search, filter) |

**Bottom icon nav** — above the system bar: bg `#FAFAFA`, p-4, subtle drop shadow, 3 equal slots with 48px Heroicons; active slot = 80px-wide `#C0D5FF` pill (radius 50). Slots: new payment (+), transactions (↑↓), more (⋮).

**Transaction card** (list screens use shadowed cards, NOT divider lists):
```
664×110 · bg #FDFDFD · border 1px #E6E8E8 · radius 8 · Level-1 shadow · 24px gap between cards
├─ 70px payment-type medallion: #F1F6FF circle + 56px Heroicon (credit-card…)
│  └─ 34px sub-badge circle (bottom-end): direction arrow (↙ payment / ↗ refund)
├─ amount Outfit Bold 25px + type label Outfit Regular 25px (stacked)
├─ StatusChip size="xl" (16px text, px-2.5 py-[5px], radius 5 — see tables.md)
├─ time Outfit Regular 25px
└─ trailing 43px arrow-in-circle affordance
```

### 4c. Key screens (Figma node ids)

- **Payment method** (`1530:2245`): app bar → amount Lead 76px → helper H5 25px → 2-col grid of squared tiles (Card / Wallet / Installment) → bottom icon nav.
- **Payment Approved / Declined** (`1736:9375` / `1736:9439`): amount 76px pinned top → 350px result illustration → H2 49px title in Success 01 / Error Base → 25px body with the amount bolded inline → 25px Bold meta (approval status + code, `#006A66`) → stacked full-width 96px buttons ("Print Customer Receipt" primary, "New Payment" secondary), 24px gap, anchored above the nav.
- **Transactions** (`1564:3720`): top row = "Reports" pill + 64px search & filter circles → "Today" section header → transaction cards → bottom icon nav (transactions slot active).
- The file also covers: splash, card/wallet/installment flows (Valu, Souhoola, Aman, InstaPay…), void, full/partial refund, receipt printing (merchant/customer), shift/full/batch reports, language & currency settings, auto settlements — pull the specific frame when implementing one of these.

### 4d. Rules of thumb

- Amounts are always the visual anchor: Lead Bold 76px navy, top-center of the screen.
- Every flow ends in a fullscreen result screen — never a toast or inline banner.
- Long-running steps (card read, printing): fullscreen state with 25px status text and a cancel action — never a small inline spinner.
- Numeric entry (amount, OTP, PIN) is its own dedicated screen; keys follow the squared-tile language (large, bordered, radius ≥8, gap-2+).

## 5. One Feature, Three Platforms (example: Transactions)

| | Web | Mobile app | POS |
|---|---|---|---|
| Shell | AppShell + Topbar | Tab bar w/ FAB + AppBar | App bar + bottom icon nav |
| Data | DataTable or RecordCards | Compact record cards (§3b) | Shadowed transaction cards (110px) |
| Filters | Filter pills / panel | Filter tabs + filter sheet | 64px search & filter circles |
| Pagination | TablePagination (compact bar) | Infinite scroll | Scroll within day sections |
| Status | StatusChip sm | StatusChip 10px + delivery mini-chips | StatusChip xl |
| Row actions | Kebab menu / detail page | Kebab → action bottom sheet | → fullscreen detail + actions |
