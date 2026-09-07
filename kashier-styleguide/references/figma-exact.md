# Figma-Exact Specs — Pixel Reproduction Reference

> **Open this file ONLY when the task is to reproduce a specific, named Figma frame pixel-for-pixel.**
> For everyday building, the principles in SKILL.md and the recipes in the other reference files take priority — these measurements are records of individual frames, not laws for every screen.
>
> Figma files: design system `EVFzgKM9DhPqmhP9F0cGdw` · Balances `lVNlJ8QUx3McAwnkJlXhHS` · Terminals `D7Fru0LdD2bDmYyXwzu9nC` · POS `nhbdiaZtsdroBgJnRbhC77` · Payment Links `Kw8SaHwCXqMlu6AbyRMeNq` · Customers `4LRW6jrxg8sgSIlgtJVN2S` · Payment Links app `D38J1PWGdciEbyJq5RrpCS` · Home-Balance app `gWH3omAqpHlhjqkAGg9zx7` · Customer Portal — Circle-pay `8Xb0ZVxoTa5ctB77GGdR85`

## Desktop topbar — Payment Links nodes `2018:43775` (test) / `3496:24627` (live), 1305×47

```
Row       : flex gap-64 items-center
Title     : Outfit Bold 39px #202020 · flex-1 · min-w-300
Search    : 370×40 · bg #F8F8F8 · border #F3F3F3 · r4 · px-16 py-8
            placeholder Noto Sans Regular 14px #919C9C · 24px search icon end
Developers: Noto Sans SemiBold 14px #001F5F · px-14 py-8 · r5
Mode chip : p-8 r4 gap-16 · label Regular 16px #513500
            test = bg #FFF7E8 border #FFE6B6 · live = bg #F3FFF4 border #4AAB4E
Switch    : 33×20 · inset track shadow 0 0 4px rgba(0,0,0,.4)
            test: track #FBAF1F/border #D9930E · knob #FFF7E8/border #B77801 (knob start)
            live: track #4AAB4E/border #388E3B · knob #F3FFF4/border #28712B (knob end)
Utilities : gap-16 · 24px question-mark-circle, bell, cog-6-tooth (outline)
Create    : 40px heroicons-solid/plus-circle · #001F5F
```

Prototype locale controls (payment-links.html) merged into the unified topbar:
```
Language : text button Noto Sans 14px #556767 ("عربي" / "EN")
Currency : chip 1px border #E6E8E8 · radius 6 · padding 6px 12px · 14px + 16px chevron
Grouping : language↔currency 12px · 24–32px to neighboring groups
```

Component code: `layout.md` → `Topbar`, `components.md` → `ModeChip`.

## Balances "Payout accounts" — node `418:12514` (1678×927) — **canonical full list page**

The reference frame for a complete portal list page: sidebar + topbar + CTA row + filter row +
table + pagination, all measured in one composition. When another file disagrees on any of these,
this frame wins.

```
PAGE GEOMETRY (viewport 1678)
  sidebar 293 + gutter 40 + content 1305 + gutter 40 = 1678
  → desktop gutter is 40px · designed content width 1305px

SIDEBAR (418:12650) — 293 wide
  inner column 261 (16px inline padding) · 24px top padding
  merchant block 70 tall (logo 38 r4 + Bold 14 name + MID chip + chevron-down) · 16px below
  nav item 44 tall, 261 wide, 48px pitch (4px gap) · 18px outline icon, 14px label
  group gap 24 · section label ("Manage") SemiBold ~15px foreground, indented 16, 8px above items

TOPBAR (418:13083) — 1305×47
  title "Accounts" Outfit Bold 39 (414 wide block)
  search 370×40 at x=431 — PILL (rounded-full), bg #F8F8F8 + 1px border,
         text inset 16, 24px magnifier 16 from the end, placeholder 16px #919C9C
  header nav 487 wide, flush to the end (x=818 → 1305):
    Developers  105×33  (text button, SemiBold 14 primary)      ← PRESENT, not omitted
    Test Mode   142×38  at +32px
    Links       104 wide at +32px — 24px question-mark-circle (outline) · bell (outline)
                · cog-6-tooth (**SOLID**), 40px pitch = 16px gaps
    Create      40px heroicons-solid/plus-circle at +32px
  → every between-group gap is 32px
  NO locale (عربي/EGP) controls in this frame — those are prototype-only

FILTER ROW (418:12737) — 1305×34, 40px above the table
  chip h34 · p-8 · 18px outline icon + 8px + 14px label + 18px heroicons-mini/chevron-down
  chip widths 159 / 142 / 191, spaced 16px · DASHED border = unset
  Export button 81×30 at the END (x=1224): 18px arrow-up-tray + SemiBold 14 primary
  → no toolbar/title inside the table card

TABLE (418:12754) — 1305×531
  header row 40 tall: label block 16 tall inset 16 from top (pt-16 pb-8), px-8
       Inter SemiBold 11 #464F60 uppercase tracking 4 (`overline`), bg Gray/0 #F7F9FC,
       border-b #E9EDF5, stacked sort carets 8px after the label
  rows 49 tall (first row 50) · cell padding 8 · zebra on even rows
  two-line cell: primary 14 #202020 at y=8 · secondary 12 Gray/500 #687182 at y=27 (2px gap)
  inline chip 8px after the primary text (PRIMARY = StatusChip positive 60×18)
  label chips h21, 8px inner padding, ~4px apart
  columns: Account 240 · Available balance 191 · Label 247 · Payout method 208 ·
           Creation date 187 · Utility 232  (= 1305)
  kebab column: 19px ellipsis-vertical; open state gets an ocean-10 circle and a
           Details/Edit dropdown (18px icons, radius 4, shadow)

PAGINATION (418:13077) — 1305×44, bg Neutral 11 #FDFDFD
  start: "1-10 of 97" 12px Gray/500
  end  : "Rows per page: 10 ⌄" · prev button (light, sd-button/inactive) · "1/10" ·
         next button (navy filled, enabled)
```

Component code: `layout.md` §5/§6b (Sidebar, Topbar) · `tables.md` §1/§3 (DataTable, pagination).

## Balances page — node `10:2413` · setup modal `1254:24866`

```
Page  : topbar → end-aligned "Manage accounts" primary → AccountSelector cards →
        Tabs (Overview/Top-ups/Activities) → KPI row → titled sections
Section: header Noto Sans Bold 16px + "Description" 12-14px muted → simple table
        (DATE | end-aligned amount + currency code)
Modal : centered ~576px card over dim/blur backdrop · illustration + Bold title +
        muted description · segmented choice buttons (selected = navy border) ·
        fields on muted panel · footer end-aligned: outline secondary + small primary
```

## Drawers — Customer Portal file `8Xb0ZVxoTa5ctB77GGdR85`

Component code: `patterns.md` §3 (`Drawer`, `DetailDrawer`, `TaskDrawer`). These are the raw pixel
measurements the component code above was derived from.

### Detail drawer — `1291:3325` → `1290:107262` (480×1080, "Transaction Details")

```
Overlay  : full-viewport scrim, drawer pinned to inline-end, full height
Header   : h56 · title x24 y16 Noto Sans Bold 16px · heroicons-solid/x-mark 24px at x432 y16
Body     : 24px inset all sides · content column 432px · 24px gap between cards
Hero     : 432×95 banner, amount centered ~39px (text-h3), status/success #4AAB4E
ID card  : border, 2 cols × 196px, 20px column gutter, 20px/16px card padding
           field = label 12px cadet (h15) → 4px → value 14px semibold (h18) · 14px between fields
Section  : title 16px bold → 8px → 1px full-width divider (border/subtle #F5F6F6) → 16px →
           2 cols × 192px, 16px gutter — repeats per section (Cashless Details, Customer Information)
Actions  : pinned bottom, 24px inset, two 208px buttons 50/50, 16px gap, h39
           outline-primary "More details" · destructive filled "Refund Transaction" (#A50017)
Vars     : radius/button 4 · border/subtle #F5F6F6 · status/success #4AAB4E · text/default #202020
```

### Task drawer — `77:1327` → `77:1403` (620×1080, "Add Customers to Portal")

```
Header   : 28px inline inset, 24px top inset · title Bold ~24px + 2-line 14px muted description
           close = heroicons-solid/x-mark 24px, aligned to title top
Context  : pill chip h≈25, 13px globe icon + 13px SemiBold label
           ("my-school.kashier.io — This Portal Only")
Tabs     : two equal 282px tabs, h53, icon circle 28px + label 14px + count badge (e.g. "1,420")
Search   : 564×39 soft-gray field, 16px magnifier at start, 14px placeholder
Filters  : "LABELS:" 11px uppercase + pill chips h≈22 (selected = filled primary), trailing "+3 more"
Selection: select-all row h32 (15px checkbox + 14px label; end caption 12px muted,
           "Showing all 1,420 customers")
Groups   : group header 12px uppercase muted, h21 (e.g. "ALL CUSTOMERS")
Row      : picker row h68 — checkbox 15px @x22 · avatar 34px @x49 · content @x95
           (name 14px semibold / meta 12px muted "phone · email · #ID" / tag chips h15)
Footer   : summary strip 564×42 tinted bg radius 4 — "Adding **4 customers**…" start /
           "Each starts at **EGP 0.00**" end
           info note = 14px icon + 2-line 12px muted
           two 276px buttons 50/50, h41, 12px gap — outline "Cancel" + primary "Add to Portal 4"
```

Both drawers share: end-anchored, full-height, own scroll region in the body, sticky footer, 24px
`heroicons-solid/x-mark` close button top-end of the header.

## Mobile APP — Payment Links app + Home-Balance app

Sections: List `21:9112` · Creation `22:9533` · Details `225:13476` · Home `461:4667`.

### Record card (app) — `911:21641` (342×116)
```
Card : bg white · border 1.5px #F8F9F9 (Neutral 10) · r4 · p-16 · rows gap-8
Row 1: StatusChip 10px uppercase (px-4 py-1 r2) + ref 12px #839090 ↔ amount Bold 14px #202020
Row 2: 18px heroicon/user + name SemiBold 14px #556767 + "Due tomorrow" 12px #839090
Row 3: indent 22 · delivery chips 0.5px border r2 px-4 py-1 · 14px icon + 10px uppercase
       Delivered #F3FFF4/#D1FFD3/#4AAB4E · Pending rgba(255,247,232,.48)/#FFE6B6/#B77801
```

### System (screenshot-verified)
```
Tab bar : 5 slots + center floating navy FAB (+) · side items 24px icon + 10px label
App bar : Bold title start · 24px search/filter/plus icons end · filter tabs below
          (All/Paid/Unpaid/Overdue underline)
Sheets  : action sheet (Details/Edit/Share/Split/Mark as paid/Clone/Delete-red)
          share sheet (QR outline btn, copy-link, send email, send SMS)
Home    : greeting (name in navy bold) + date + bell-with-badge · Services 2-col tile grid
          · Reports: scope chips + muted stat card (uppercase label + big amount)
          + compact tabs + txn rows + "Go to balance →" outline
States  : illustrated empty + CTA · no-results + "Clear search/filters" · skeletons
          · coming-soon (icon circle + title + orange chip + description)
```

Patterns: `platforms.md` §3.

## Alert — node `96:1292`

| Type | Background | Border | Text |
|------|-----------|--------|------|
| Error | `#FFEAED` (Error 09) | `#FFBBC4` (Error 08) | `#A50017` (Error Base) |
| Success | `#F3FFF4` (Success 09) | `#AEFFB2` (Success 07) | `#1A541D` (Success 01) |
| Warning | `#FFF7E8` (Warning 09) | `#FFD484` (Warning 07) | `#513500` (Warning 01) |
| Info | `#F1F6FF` (Ocean 09) | `#C0D5FF` (Ocean 07) | `#001F5F` (Primary) |

- `border: 1px solid` (full border — NOT left-only)
- `border-radius: 5px` (NOT the global 4px)
- `padding: 12px`, `gap: 8px`
- Icon: 24px Heroicons outline, `flex-shrink-0`; left in LTR, right in RTL (`flex-direction: row-reverse`)
- Font: Noto Sans 14px (LTR) · Noto Naskh Arabic 14px (RTL); title 700, message 400, same color for both
- Icons: Error → `ExclamationCircleIcon`, Success → `CheckCircleIcon`, Warning → `ExclamationTriangleIcon`, Info → `InformationCircleIcon`

## Card — node `174:8655` (Balances file)

| Property | Value | Token |
|----------|-------|-------|
| Border | `1px solid #F5F6F6` | `--border-subtle` (Neutral 09) |
| Border radius | `4px` | `--radius` |
| Padding | `16px` | `--space-4` |
| Section gap | `24px` | `--space-6` |
| Header / internal dividers | `1px solid #F5F6F6` | `--border-subtle` |
| Header title font | Outfit Bold 25px | `font-display text-h5 font-bold` |
| Header title color | `#556767` | `text-cadet-base` |
| Section label | Noto Sans Bold 16px `#556767` | `font-body text-base font-bold text-cadet-base` |
| Row label | Noto Sans Regular 14px `#556767` | `font-body text-xs font-normal text-cadet-base` |
| Row value | Noto Sans SemiBold 16px `#202020` | `font-body text-base font-semibold text-foreground` |

## Sidebar — Balances file node `10:2549`

```
Background  : #F8F9F9 (Neutral 10)
Width       : 261px content + 16px padding both sides = 293px total
Logo area   : avatar 38×38 radius 4px + Bold 14px #202020 company name
Nav item    : px-4 py-3 (16px/12px) · radius 8px · flex items-center gap-2
Icon        : 20px Heroicons outline · color #001F5F (primary)
Label default: Noto Sans Regular 14px #001F5F
Label active : Noto Sans Regular 14px #556767 + arrow icon visible (right)
Hover       : bg rgba(0,31,95,0.06)
Item gap    : gap-1 (4px) within group
Group gap   : gap-6 (24px) between groups
Section label: SemiBold 12px #556767 · uppercase · tracking-[0.96px] · px-4
```

Component code: `layout.md` → `Sidebar`.

## Account Selector — Balances file node `183:7867`

```
Card padding    : 16px
Card radius     : 4px (var(--radius))
Card gap        : 16px between cards
Selected border : 1px solid #001F5F (primary)
Default border  : 1px solid #E6E8E8 (--border)
Account name    : Noto Sans SemiBold 14px #556767
Sub label       : Noto Sans Regular 12px #839090 (Neutral 04)
Balance default : Noto Sans Bold 16px #556767
Balance selected: Noto Sans Bold 16px #202020  ← brightest when active
```

Component code: `components.md` → `AccountSelector`.

## KPI Widget Bar — Balances file node `10:4785`

```
Container   : flex row, single 1px dividers between cells (implement as container
              border + gap-px — NEVER per-cell borders, they double up)
Widget bg   : #FFFFFF
Widget border: 1px solid #F5F6F6 (--border-subtle)
First widget: rounded-l (4px tl/bl) · Last widget: rounded-r (4px tr/br)
Padding     : 16px
Label       : Noto Sans Regular 14px #556767 UPPERCASE
Value       : Outfit Bold 31px #202020
Description : Noto Sans Regular 12px #B3BBBB (Neutral 06)
```

Component code: `components.md` → `KpiBar`.

## Data Table — Balances file node `10:3463`

```
Container   : bg white · rounded · overflow-hidden
Shadow      : 0 0 0 1px rgba(152,161,178,0.1),
              0 1px 4px 0 rgba(69,75,87,0.12),
              0 0 2px 0 rgba(0,0,0,0.08)
Header bg   : rgba(245,246,246,0.75) · backdrop-blur-[4px]
Header border: border-bottom 1px solid #E9EDF5 · height 40px · px 8px
Header text : Inter SemiBold 11px · #464F60 · uppercase · tracking-[0.44px]
Row height  : 49px · px 8px
Even rows   : bg rgba(248,249,249,0.75)
Cell text   : Noto Sans Regular 14px #202020
Amount col  : text-right · amount #202020 14px + currency code #687182 12px
```

Component code: `tables.md` → `DataTable` (v2 keeps this visual spec and adds sorting, states, pagination).

## Terminals — Table, Pagination, Status (Terminals file, frame `151:7793`)

### Table — node `151:8212`
```
Header      : pt-16 pb-8 px-8 · bg rgba(245,246,246,0.75) + backdrop-blur 4px
              border-b #E9EDF5 · Inter SemiBold 11px #464F60 uppercase tracking 0.44px
Sort        : stacked carets 7×5px · inactive #BCC2CE · active #464F60
Rows        : 56px · p-8 · zebra even rgba(248,249,249,0.75)
Select col  : leading 32px · 16px checkboxes
Utility col : trailing 35px · 19px ellipsis-vertical
```

### Table pagination — node `151:8600` (1301×44)
```
Bar      : px-20 py-12 · gap-20 · bg #FDFDFD (Neutral 11) · backdrop-blur 4px · rounded-b 4px
Captions : Inter Medium 12px #687182 · tracking 0.36px
Layout   : "1-10 of 97" (start, flex-1) · "Rows per page: 10 ▾" · [<] 1/10 [>]
Buttons  : px-4 py-2 · radius 6 · 16px icon
           enabled  bg white + 0 1px 1px rgba(0,0,0,.1), 0 0 0 1px rgba(70,79,96,.16),
                    0 2px 5px rgba(89,96,120,.1)
           disabled bg #F7F9FC + 0 0 0 1px rgba(70,79,96,.24)
Fraction : current #171C26 · "/total" #687182
No numbered page buttons.
```

### Status chip — nodes `151:8361` (inactive) / `151:8363` (active)
```
Size    : 18px tall · px-4 py-2 · radius 2
Text    : Noto Sans Regular 12px · uppercase · leading 1.2
Active  : bg #D1FFD3 (Success 08) · text #1A541D (Success 01)
Inactive: bg #FFEAED (Error 09) · text #A50017 (Error Base)
XL (POS "StatusXLarge"): 16px text · px-10 py-5 · radius 5 · same colors
```

### Filter chips — Payment Links `2226:49748` (canonical), Terminals `151:8184`
```
Chip : radius 4 · p-8 · gap-8 · 18px Heroicon + Noto Sans Regular 12px + 18px chevron-down
State: EMPTY  = border 1px DASHED #D5D9D9 · label #556767
       FILLED = border 1px SOLID #D5D9D9 · label shows applied value ("Payment status: Paid")
Row  : gap-16 · sits between topbar and data · end side may carry 30px buttons
```

Component code: `tables.md` → `TablePagination`, `StatusChip`, `FilterChip`.

## Entity list cards (RecordCard)

### Customers record — `506:9868` (desktop 1301×119) · group `506:9865` · full frame `505:15539`
```
Card      : bg white · border 1px #F5F6F6 · radius 4 · p-16 · gap-16 between cards
Group     : "Today" Outfit Bold 25px black · 16px header→cards · 24px between groups
Identity  : 18px user icon + name Noto Sans SemiBold 14px #202020 + ref 12px #839090 (gap 2)
            fixed w-300 min-w-250
Attributes: 18px icon (at-symbol / device-phone-mobile) + SemiBold 12px #556767
            flex-1 min-w-150 max-w-310 · row is flex-wrap gap-8
Chips     : pill radius 16 · bg #F5F6F6 · border #E6E8E8 · ps-8 pe-4 py-2 · Regular 14px #202020
Kebab     : absolute end, top -12 · 39px hit · 19px ellipsis-vertical · rounded-40
Meta      : border-t #F5F6F6 pt-8 · flex-wrap gap-8 · Item = label Regular 12px #839090
            over value SemiBold 12px #556767 (gap 2) · flex-1 min-w-150
Annotation: "On hover select box should be visible" — checkbox 16px radius 4 border #D5D9D9,
            checked = #001F5F border + navy 12px check (outline, not filled)
```

### Payment Links card — `2237:44002` (1301×108)
```
Same card shell. Differences:
Title     : Noto Sans SemiBold 16px #556767 (leading-none) + ref Regular 14px #839090
Amount    : Noto Sans Bold 16px #202020 + StatusChip inline (PAID: #D1FFD3/#1A541D, p-4 r-2)
Notify    : 18px envelope-open / chat-bubble icons with 8px check-circle overlay (bottom-end)
Meta      : horizontal items — 16px icon + Regular 14px #839090 text · min-w-150
            may embed neutral StatusChip (EXPIRED: bg #F5F6F6 text #556767)
Checkbox  : visible at start of title row when selection active
```

### Customers mobile — `455:9921` (396-wide screen)
```
Top bar   : 50px row — account switcher (38px logo r4 + Noto Sans Bold 14px + 20px chevron-down)
            · end: 24px bell + 32px bars-3-bottom-right
Page      : title Outfit Bold 25px → full-width primary Button h-48 r4
            (24px plus + SemiBold 16px white)
Search    : bg #F8F8F8 border #F3F3F3 r4 · ps-16 pe-8 py-8 · placeholder Regular 14px #556767
            · 24px search icon end
Date chip : dashed border #D5D9D9 r4 p-8 · 18px calendar icon + Regular 12px #556767
Select all: 14px check + SemiBold 12px #001F5F · px-12 py-8
Groups    : headers Outfit Bold 20px · 8px header→cards, 8px card gap · 24px between groups
Cards     : identical RecordCard anatomy; meta wraps to 2 cols via min-w-150; chips at bottom
```

Component code: `lists.md` → `RecordCard` family.

## POS flows (POS file, "User Flow" page `1527:1685`)

Device 720×1440 · content 664 wide (28px margins) · Android status bar 48px + system nav 96px.
Full system spec: `platforms.md` §4. Key frames:

| Screen | Node | Highlights |
|--------|------|-----------|
| Payment method | `1530:2245` | App bar bg #F8F9F9, title Outfit Bold 31 · amount Lead Bold 76 #001F5F · squared tiles 322×304 r25 border-2 #103788 shadow 0 0 10px rgba(0,0,0,.25) · bottom icon nav bg #FAFAFA, active pill #C0D5FF 80px |
| Payment Approved | `1736:9375` | 76px amount top · 350px illustration · H2 Bold 49 #1A541D title · 25px body · meta 25 Bold #006A66 · buttons 664×96 r8 (primary navy solid / secondary navy border, 50px icon) |
| Payment Declined | `1736:9439` | Same skeleton, title/illustration in Error colors |
| Transactions | `1564:3720` | "Reports" pill r44 px-24 py-16 25px · 64px #F1F6FF utility circles w/ 32px icons · "Today" Outfit Bold 31 #697979 · cards 664×110 bg #FDFDFD border #E6E8E8 r8 Level-1 shadow · 70px medallion (#F1F6FF, 56px icon, 34px direction sub-badge) · text 25px · StatusXLarge chip · 43px trailing arrow |
| Splash | `1618:3004` | Entry screen |

Other flows in the file: card/wallet/installment (Valu `2662:14504`, Souhoola `3934:7489`, Aman `4324:8257`, InstaPay section `5230:4883`), void, full/partial refund, receipt printing, shift/full/batch reports, language/currency, auto settlements (`5843:8856+`).
