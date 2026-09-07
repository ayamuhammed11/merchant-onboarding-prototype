# Kashier Design Tokens — Full Reference

> Source of truth: Figma file `EVFzgKM9DhPqmhP9F0cGdw`.
> Load this file when setting up a project (globals.css / tailwind.config) or when you need a primitive palette value not covered by the semantic cheat sheet in SKILL.md.

## 1. CSS Variables (paste into `globals.css`)

```css
@layer base {
  :root {
    /* ── Semantic (use these in components) ── */
    --background:             0 0% 100%;
    --foreground:             0 0% 13%;          /* #202020 */
    --card:                   0 0% 100%;
    --card-foreground:        0 0% 13%;
    --popover:                0 0% 100%;
    --popover-foreground:     0 0% 13%;

    /* Primary = Ocean (navy) */
    --primary:                218 100% 19%;      /* #001F5F */
    --primary-foreground:     0 0% 100%;

    /* Secondary = Peachy */
    --secondary:              21 100% 71%;       /* #FF9D6C */
    --secondary-foreground:   21 60% 19%;        /* #502712 */

    /* Accent = Kashier Teal */
    --accent:                 177 100% 37%;      /* #00BCB4 */
    --accent-foreground:      0 0% 100%;

    --muted:                  180 6% 96%;        /* #F5F6F6 */
    --muted-foreground:       178 8% 44%;        /* #637373 */

    --border:                 180 5% 89%;        /* #E6E8E8 — Neutral 08, inputs / table rows / strong dividers */
    --border-subtle:          180 6% 96%;        /* #F5F6F6 — Neutral 09, card borders / light dividers */
    --input:                  180 5% 89%;
    --ring:                   218 100% 19%;      /* #001F5F */

    /* Semantic status */
    --destructive:            349 100% 32%;      /* #A50017 */
    --destructive-foreground: 0 0% 100%;
    --warning:                41 96% 55%;        /* #FBAF1F */
    --warning-foreground:     37 100% 16%;       /* #513500 */
    --success:                123 39% 48%;       /* #4AAB4E */
    --success-foreground:     0 0% 100%;

    /* Radius */
    --radius: 0.25rem;   /* 4px — Figma default */

    /* ── Aliases used by the HTML prototypes (map to primitives) ── */
    --primary-hover: #103788;   /* Ocean 90 — hover state of primary buttons/links */
    --link:          #2955B1;   /* Ocean 80 — text links, linked table cells (id, date) */
    --primary-10:    #F1F6FF;   /* Ocean 10 — row hover, active page button bg, chips */
    --bg:            #F8F9F9;   /* Neutral 10 — sidebar background */

    /* ── Table / pagination gray palette (Figma DS, Inter context) ── */
    --table-gray-700: #464F60;   /* header text, pager icons */
    --table-gray-500: #687182;   /* pagination captions, currency codes */
    --table-gray-400: #868FA0;   /* disabled pager icon */
    --table-gray-200: #BCC2CE;   /* inactive sort carets */
    --table-gray-0:   #F7F9FC;   /* disabled pager button bg */
    --table-ink:      #171C26;   /* current page number */
    --neutral-11:     #FDFDFD;   /* pagination bar bg, POS cards bg */
    --success-08:     #D1FFD3;   /* StatusChip positive bg */
  }
}
```

> **POS Figma naming note:** in the POS file (`nhbdiaZtsdroBgJnRbhC77`) the navy scale is called **"Accent"** — Accent Base `#001F5F`, Accent 01 `#103788`, Accent 07 `#C0D5FF`, Accent 09 `#F1F6FF`. These are the **Ocean** palette (ocean-100/90/30/10 equivalents), not the teal `--accent`. Never invent a second navy palette from that naming.

## 2. Tailwind Config Extension (`tailwind.config.ts`)

```ts
import { type Config } from "tailwindcss";

export default {
  theme: {
    extend: {
      screens: {
        "3xl": "1920px",   /* layout.md §3 — list/dashboard container caps here. Under `extend`
                               so it ADDS to the defaults (sm/md/lg/xl/2xl) instead of replacing them. */
      },
      maxWidth: {
        content:        "1600px",   /* list/dashboard AppShell tier — layout.md §2 */
        "content-detail": "1305px", /* detail tier = canonical Figma content width */
      },
      spacing: {
        sidebar: "293px",           /* canonical sidebar width (Balances 418:12650) */
      },
      colors: {
        border:         "hsl(var(--border))",
        "border-subtle":"hsl(var(--border-subtle))",
        input:       "hsl(var(--input))",
        ring:        "hsl(var(--ring))",
        background:  "hsl(var(--background))",
        foreground:  "hsl(var(--foreground))",
        primary: {
          DEFAULT:    "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT:    "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        accent: {
          DEFAULT:    "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        destructive: {
          DEFAULT:    "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        warning: {
          DEFAULT:    "hsl(var(--warning))",
          foreground: "hsl(var(--warning-foreground))",
        },
        success: {
          DEFAULT:    "hsl(var(--success))",
          foreground: "hsl(var(--success-foreground))",
        },
        muted: {
          DEFAULT:    "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        card: {
          DEFAULT:    "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        /* Primitive palettes (use sparingly — prefer semantic) */
        kashier: {
          100: "#006A66", 90: "#00938D", base: "#00BCB4",
          70: "#15E5DC",  60: "#2FFFF6", 50: "#54FFF7",
          40: "#7AFFF9",  30: "#9FFFFB", 20: "#C5FFFC", 10: "#EAFFFE",
        },
        ocean: {
          100: "#001F5F", 90: "#103788", 80: "#2955B1",
          70: "#4C7AD9",  60: "#77A3FF", 50: "#8FB4FF",
          40: "#A8C4FF",  30: "#C0D5FF", 20: "#D9E5FF", 10: "#F1F6FF",
        },
        watermelon: {
          100: "#61000D", 90: "#830012", base: "#A50017",
          70: "#C70C25",  60: "#E91C38", 50: "#FF2E4B",
          40: "#FF5D73",  30: "#FF8C9C", 20: "#FFBBC4", 10: "#FFEAED",
        },
        lemon: {
          100: "#1A541D", 90: "#28712B", 80: "#388E3B",
          base: "#4AAB4E", 60: "#5EC863", 50: "#75E57A",
          40: "#8CFF91",  30: "#AEFFB2", 20: "#D1FFD3", 10: "#F3FFF4",
        },
        mango: {
          100: "#513500", 90: "#734B00", 80: "#956200",
          70: "#B77801",  60: "#D9930E", base: "#FBAF1F",
          40: "#FFC352",  30: "#FFD484", 20: "#FFE6B6", 10: "#FFF7E8",
        },
        cadet: {
          black: "#202020",
          base:  "#556767", "01": "#637373", "02": "#697979",
          "03":  "#6F7E7E", "04": "#839090", "05": "#919C9C",
          "06":  "#B3BBBB", "07": "#D5D9D9", "08": "#E6E8E8",
          "09":  "#F5F6F6", white: "#FFFFFF",
        },
      },
      fontFamily: {
        display:        ["Outfit", "sans-serif"],
        body:           ["Noto Sans", "sans-serif"],
        "display-ar":   ["Cairo", "sans-serif"],
        "body-ar":      ["Noto Naskh Arabic", "sans-serif"],
      },
      fontSize: {
        /* English type scale */
        "lead":   ["76px", { lineHeight: "1.2", fontWeight: "700" }],
        "h1":     ["61px", { lineHeight: "1.2" }],
        "h2":     ["49px", { lineHeight: "1.2" }],
        "h3":     ["39px", { lineHeight: "1.2" }],
        "h4":     ["31px", { lineHeight: "1.2" }],
        "h5":     ["25px", { lineHeight: "1.2" }],
        "h6":     ["20px", { lineHeight: "1.2" }],
        "base":   ["16px", { lineHeight: "1.2" }],
        "xs":     ["14px", { lineHeight: "1.2" }],
        "xxs":    ["12px", { lineHeight: "1.2" }],
        /* Arabic xSmall is 13px (not 14), xxSmall is 10px (not 12) */
        "xs-ar":  ["13px", { lineHeight: "1.2" }],
        "xxs-ar": ["10px", { lineHeight: "1.2" }],
      },
      borderRadius: {
        sm:   "2px",
        DEFAULT: "4px",   /* --radius */
        md:   "4px",
        lg:   "8px",
        full: "9999px",
      },
      boxShadow: {
        sm:  "0 1px 2px 0 rgb(0 0 0 / 0.05)",
        md:  "0 0 5px 0 rgb(0 0 0 / 0.25)",  /* Figma Level 1 */
        lg:  "0 4px 16px 0 rgb(0 0 0 / 0.12)",
      },
    },
  },
} satisfies Config;
```

## 3. Typography Rules

| Role | EN Font | AR Font | Weight options |
|------|---------|---------|----------------|
| Display / H1–H6 | **Outfit** | **Cairo** | 400, 700 |
| Body / UI text | **Noto Sans** | **Noto Naskh Arabic** | 400, 600, 700 |

```tsx
// Heading — English
<h1 className="font-display text-h1 font-bold text-foreground">Payment Platform</h1>

// Body text — English
<p className="font-body text-base font-normal text-foreground">Your transaction is secure.</p>

// Heading — Arabic (RTL)
<h1 className="font-display-ar text-h1 font-bold text-foreground" dir="rtl">منصة المدفوعات الرقمية</h1>

// Body — Arabic
<p className="font-body-ar text-base font-normal text-foreground" dir="rtl">تتم معالجة مدفوعات التاجر بشكل آمن.</p>
```

> **AR size exception**: xSmall = 13px (`text-xs-ar`), xxSmall = 10px (`text-xxs-ar`)
> **Lead Regular** has `leading-none` (1.0), all others use `leading-[1.2]`.

## 4. Extended Token Table

| Token | Value | Use for |
|-------|-------|---------|
| `--primary` | #001F5F Ocean | Buttons, links, focus rings, active nav |
| `--accent` | #00BCB4 Teal | Accent buttons, highlights, progress |
| `--secondary` | #FF9D6C Peachy | Secondary actions, warm highlights |
| `--destructive` | #A50017 | Delete, error actions |
| `--warning` | #FBAF1F | Warning states, pending badges |
| `--success` | #4AAB4E | Paid, verified, completed |
| `--border` | #E6E8E8 (Neutral 08) | Input borders, table rows, account cards unselected |
| `--border-subtle` | #F5F6F6 (Neutral 09) | **Card borders**, KPI widget borders, header dividers, internal row dividers |
| `--muted` | #F5F6F6 | Card footers bg, disabled bg, muted surfaces |
| `--muted-foreground` | #637373 (Neutral 01) | Hints, placeholders, secondary text |
| `cadet-base` | #556767 (Neutral Base) | Card titles, section labels, row labels, nav labels |
| `#839090` Neutral 04 | — | Account sub-labels |
| `#B3BBBB` Neutral 06 | — | KPI widget descriptions |
| `#464F60` | — | Table header text |
| `#687182` | — | Currency code text in tables |
| Sidebar bg | #F8F9F9 Neutral 10 | App shell sidebar |
| Content bg | #FFFFFF | Main content area |
| `--link` | #2955B1 (Ocean 80) | Text links, linked table cells |
| `--primary-hover` | #103788 (Ocean 90) | Primary button/link hover |
| `--primary-10` | #F1F6FF (Ocean 10) | Row hover, chips, POS icon circles |
| Gray/700 | #464F60 | Table header text, pager icons |
| Gray/500 | #687182 | Pagination captions, currency codes |
| Gray/0 | #F7F9FC | Disabled pager button bg |
| Neutral 11 | #FDFDFD | Pagination bar bg, POS card bg |
| Success 08 | #D1FFD3 | StatusChip positive bg |
| sd-button shadow | 0 1px 1px rgba(0,0,0,.1), 0 0 0 1px rgba(70,79,96,.16), 0 2px 5px rgba(89,96,120,.1) | Enabled pager buttons |
| sd-button inactive | 0 0 0 1px rgba(70,79,96,.24) | Disabled pager buttons |
| `--radius` | 4px | All corners (alerts: 5px) |
| Heroicons outline | 20px nav · 24px alerts/standalone | All icons |
| `shadow-md` | 0 0 5px rgba(0,0,0,0.25) | Raised cards |
| `screens.3xl` | 1920px | A 1920 monitor still fills; the container caps just past it (layout.md §2–3) |
| `maxWidth.content` | 1600px | List/dashboard `AppShell size="wide"` container |
| `maxWidth.content-detail` | 1305px | Detail-page container — the canonical Figma content width |
| `spacing.sidebar` | 293px | Canonical sidebar width (Balances `418:12650`) |
| Desktop gutter | 40px (`xl:p-10`) | Canonical page padding either side of the content |
| Table row / header | 49px / 40px | Canonical DataTable metrics (tables.md §1) |
| `overline` | Inter SemiBold 11px, tracking 4 | Table column headers |
| Drawer sizes | 400 / 480 / 620 / 760px | `sm`/`md`/`lg`/`xl` drawer widths (patterns.md §3) |
| `shadow-lg` | 0 4px 16px rgba(0,0,0,0.12) | Drawer/modal panels |
