# Kashier Tables — DataTable, Pagination, States, Responsive

> Load this file whenever a screen shows tabular data.
> A table is not done until it has: pagination, an empty state, a loading state, and a `<md` strategy.

## 1. DataTable

Canonical visual spec — **Balances "Payout accounts" full page, node `418:12514`** (file `lVNlJ8QUx3McAwnkJlXhHS`), cross-checked against Terminals `151:8212`. This is the reference frame for a complete list page: sidebar + topbar + page action + filter row + table + pagination, all measured together (`figma-exact.md` → *Balances Payout accounts*).

```
Container : bg white · rounded · overflow hidden · width = content container (1305 canonical)
Toolbar   : NONE in the canonical list page — there is no title/search bar inside the card.
            Page-level actions live OUTSIDE it: the "Export" text button sits at the END of
            the FILTER ROW above the card, and the primary CTA ("New Account") is its own
            end-aligned row above that. Only add a TableToolbar when a page carries two or
            more tables that each need naming (e.g. Balances §7d-bis titled sections).
Header    : h-10 (40px) · px-2 · label block 16px tall inset 16px from the top (pt-4 pb-2)
            bg Gray/0 #F7F9FC · border-b #E9EDF5
            Inter SemiBold 11px #464F60 uppercase · tracking 4 (the `overline` token)
            sort: stacked carets 8px after the label, inactive #BCC2CE, active #464F60
Rows      : h-[49px] (first row 50px) · p-2 (8px) · align middle
            zebra: even rows bg rgba(248,249,249,0.75) · hover bg ocean-10
Cells     : single-line  → Noto Sans 14px #202020
            TWO-LINE (the canonical default for identity/method columns):
              primary 14px #202020 at y=8 · secondary 12px Gray/500 #687182 at y=27 (2px gap)
              e.g. "Misr Gadida 1 branch" / "ACCXXXX" · "Bank Account" / "CIB ****1234"
            inline chip: 8px after the primary text (e.g. PRIMARY StatusChip 60×18)
Columns   : natural widths, sum = container. Canonical Accounts table (1305 total):
            Account 240 · Available balance 191 · Label 247 · Payout method 208 ·
            Creation date 187 · Utility (kebab) 232
Select col: optional leading 32px column with 16px checkboxes
Utility col: trailing column · 19px ellipsis-vertical kebab → dropdown menu (Details / Edit,
            18px icons, radius 4, shadow-lg, ~180px). Open kebab gets an ocean-10 circle.
Link cells: record id 600 / date 400 in --link (#2955B1, Ocean 80)
Status    : StatusChip (§ below) — 18px tall, radius 2, Noto Sans Regular 12px uppercase
Scroll    : overflow-x-auto wrapper · table min-w to natural width
Pagination: compact bar (§3), h-11 (44px) — count at START · rows-per-page + prev + fraction
            + next at the END
```

```tsx
// components/ui/DataTable.tsx
import { ChevronUpIcon, ChevronDownIcon, ChevronUpDownIcon } from "@heroicons/react/24/outline";
import { cn } from "@/lib/utils";
import { TablePagination, type TablePaginationProps } from "./TablePagination";

export type Column<T> = {
  key: string;
  header: string;
  align?: "start" | "end";
  width?: string;                            // e.g. "w-32", "min-w-[160px]"
  sortable?: boolean;
  render?: (row: T) => React.ReactNode;
};

export type SortState = { key: string; dir: "asc" | "desc" } | null;

export function DataTable<T extends { id?: string | number }>({
  columns,
  rows,
  loading = false,
  emptyState,
  sort,
  onSortChange,
  onRowClick,
  stickyHeader = false,
  toolbar,
  pagination,
}: {
  columns: Column<T>[];
  rows: T[];
  loading?: boolean;
  emptyState?: React.ReactNode;              // <EmptyState …/> — required when rows can be empty
  sort?: SortState;
  onSortChange?: (s: SortState) => void;
  onRowClick?: (row: T) => void;
  stickyHeader?: boolean;
  toolbar?: React.ReactNode;                 // <TableToolbar …/> — title + search + filter/export links
  pagination?: TablePaginationProps;         // renders the footer bar when provided
}) {
  const toggleSort = (key: string) => {
    if (!onSortChange) return;
    if (sort?.key !== key) onSortChange({ key, dir: "asc" });
    else if (sort.dir === "asc") onSortChange({ key, dir: "desc" });
    else onSortChange(null);
  };

  return (
    <div className="bg-white rounded border border-border-subtle overflow-hidden">
      {toolbar}
      {/* Horizontal scroll is the default overflow strategy — table keeps its natural width */}
      <div className="overflow-x-auto">
        <table className="w-full border-collapse min-w-[640px]">
          <thead>
            <tr>
              {columns.map(col => (
                <th
                  key={col.key}
                  aria-sort={sort?.key === col.key ? (sort.dir === "asc" ? "ascending" : "descending") : undefined}
                  className={cn(
                    "h-10 pt-4 pb-2 px-2 border-b border-[#E9EDF5] bg-[#F7F9FC]",
                    "font-[Inter,sans-serif] text-[11px] font-semibold text-[#464F60] uppercase tracking-[4px] leading-4 whitespace-nowrap align-bottom",
                    col.align === "end" ? "text-end" : "text-start",
                    col.width,
                    stickyHeader && "sticky top-0 z-10",
                  )}
                >
                  {col.sortable && onSortChange ? (
                    <button
                      onClick={() => toggleSort(col.key)}
                      className={cn(
                        "inline-flex items-center gap-1 uppercase hover:text-primary transition-colors",
                        sort?.key === col.key && "text-primary",
                      )}
                    >
                      {col.header}
                      {sort?.key === col.key
                        ? (sort.dir === "asc" ? <ChevronUpIcon className="w-3 h-3" /> : <ChevronDownIcon className="w-3 h-3" />)
                        : <ChevronUpDownIcon className="w-3.5 h-3.5 opacity-50" />}
                    </button>
                  ) : col.header}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {loading ? (
              <SkeletonRows columns={columns.length} />
            ) : rows.length === 0 ? (
              <tr>
                <td colSpan={columns.length} className="p-0">
                  {emptyState ?? <EmptyState title="No results" />}
                </td>
              </tr>
            ) : (
              rows.map((row, i) => (
                <tr
                  key={row.id ?? i}
                  onClick={onRowClick ? () => onRowClick(row) : undefined}
                  className={cn(
                    i % 2 === 1 && "bg-[rgba(248,249,249,0.75)]",
                    onRowClick && "cursor-pointer hover:bg-ocean-10 transition-colors",
                  )}
                >
                  {columns.map(col => (
                    <td
                      key={col.key}
                      className={cn(
                        "h-[49px] p-2 text-xs font-normal text-foreground font-body align-middle whitespace-nowrap",
                        col.align === "end" ? "text-end" : "text-start",
                      )}
                    >
                      {col.render ? col.render(row) : String((row as Record<string, unknown>)[col.key] ?? "")}
                    </td>
                  ))}
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>
      {pagination && <TablePagination {...pagination} />}
    </div>
  );
}

// Loading skeleton — always 5 rows so the layout doesn't jump
function SkeletonRows({ columns }: { columns: number }) {
  return (
    <>
      {Array.from({ length: 5 }).map((_, r) => (
        <tr key={r} className={r % 2 === 1 ? "bg-[rgba(248,249,249,0.75)]" : ""}>
          {Array.from({ length: columns }).map((_, c) => (
            <td key={c} className="h-[49px] p-2">
              <div className="h-3.5 rounded bg-cadet-09 animate-pulse" style={{ width: `${55 + ((r * 7 + c * 13) % 35)}%` }} />
            </td>
          ))}
        </tr>
      ))}
    </>
  );
}
```

### TableToolbar (the exception, not the default)

> **The canonical list page has NO toolbar inside the table card** (Balances `418:12514`). Export
> is a text button at the end of the filter row, the primary CTA is its own end-aligned row, and
> the table card starts directly with the header row. Use `TableToolbar` ONLY when a page shows
> several tables that each need a name (Balances titled sections, `layout.md` §7d-bis).
>
> **One search per page.** When the page has the topbar module search (layout.md §6b) — which every portal list page does — do NOT render a second search in the table toolbar; pass no `onSearch` so the toolbar is just title + Filter/Export links. The toolbar search exists only for pages without a topbar search (e.g. a table inside a detail page).

```tsx
// Toolbar inside the table card: title + end-side links (+ search ONLY if no topbar search)
export function TableToolbar({ title, search, onSearch, children }: {
  title: string;
  search?: string;
  onSearch?: (q: string) => void;
  children?: React.ReactNode;      // toolbar links: Filter, Export…
}) {
  return (
    <div className="flex items-center gap-4 p-4 flex-wrap">
      <h2 className="font-body text-[18px] font-semibold text-cadet-base shrink-0">{title}</h2>
      {onSearch && (
        <div className="relative flex-1 min-w-60">
          <MagnifyingGlassIcon className="w-4 h-4 absolute start-1.5 top-1/2 -translate-y-1/2 text-cadet-05" />
          <input
            value={search}
            onChange={e => onSearch(e.target.value)}
            placeholder="Search…"
            className="w-full h-[38px] rounded border-0 bg-transparent ps-8 pe-3 font-body text-[13px] text-foreground placeholder:text-cadet-05 focus:outline-none"
          />
        </div>
      )}
      <div className="flex items-center gap-5 ms-auto">{children}</div>
    </div>
  );
}

// Toolbar link: 13px semibold, icon in cadet-base
export function ToolbarLink({ icon: Icon, children, ...props }: React.ButtonHTMLAttributes<HTMLButtonElement> & { icon?: React.ComponentType<{ className?: string }> }) {
  return (
    <button className="flex items-center gap-1.5 text-[13px] font-semibold text-foreground hover:text-primary" {...props}>
      {Icon && <Icon className="w-4 h-4 text-cadet-base" />}
      {children}
    </button>
  );
}
```

### Filter row (canonical — Balances `418:12737`, also `2226:49748` / Terminals `151:8184`)

The filter row is a **34px-tall strip**: filter chips at the start, the **Export text button at the
end**. Chips are **radius 4 (NOT a pill)**, `h-[34px] p-2`, 18px leading Heroicon + 8px gap +
Noto Sans **14px** label + 18px `chevron-down` at the end, spaced **16px** apart. **The filter row
OWNS the table below it: wrap both in a `gap-3` (12px) sub-stack** — canonical measures 40px, so
`gap-8`/`gap-10` is equally valid — separated from other page regions by the page's 32px gap.

**Border encodes state: dashed = empty (no filter set) · solid full border = filter applied.** A filled chip shows its value in the label. The prototypes' expanding filter panel remains a valid alternative for complex multi-field filtering.

```tsx
export function FilterChip({ icon: Icon, label, value, ...props }: React.ButtonHTMLAttributes<HTMLButtonElement> & {
  icon: React.ComponentType<{ className?: string }>;
  label: string;                 // "Select a method", "Select a label", "Select a value date range"…
  value?: string;                // applied value → solid border + value shown ("Payment status: Paid")
}) {
  const filled = Boolean(value);
  return (
    <button
      className={cn(
        "inline-flex items-center gap-2 h-[34px] p-2 rounded border bg-white font-body text-xs transition-colors",
        filled
          ? "border-solid border-cadet-07 text-foreground"
          : "border-dashed border-cadet-07 text-cadet-base hover:border-cadet-05",
      )}
      {...props}
    >
      <Icon className="w-[18px] h-[18px] shrink-0" />
      <span className="whitespace-nowrap">
        {label}{filled && <>: <span className="font-semibold">{value}</span></>}
      </span>
      <ChevronDownIcon className="w-[18px] h-[18px] shrink-0" />
    </button>
  );
}

// Canonical filter row: chips at the start, Export at the end. This row sits OUTSIDE the
// table card (common-region law) and directly above it.
<div className="flex items-center gap-4 flex-wrap">
  <FilterChip icon={CheckCircleIcon} label="Select a method" />
  <FilterChip icon={TagIcon} label="Select a label" value="VIP" />   {/* applied → solid border */}
  <FilterChip icon={CalendarDaysIcon} label="Select a value date range" />
  <button className="ms-auto flex items-center gap-1.5 font-body text-xs font-semibold text-primary hover:text-ocean-90">
    <ArrowUpTrayIcon className="w-[18px] h-[18px]" /> Export
  </button>
</div>
```

### Cell helpers

```tsx
// TwoLineCell — the canonical identity cell (Balances 418:12801). Primary 14px + muted 12px
// secondary underneath, 2px apart. Use for account/customer/method columns; `chip` renders an
// inline StatusChip 8px after the primary text (the "PRIMARY" badge in the reference frame).
export function TwoLineCell({ primary, secondary, chip }: {
  primary: React.ReactNode;
  secondary?: React.ReactNode;
  chip?: React.ReactNode;
}) {
  return (
    <div className="flex flex-col gap-[2px] min-w-0">
      <div className="flex items-center gap-2 min-w-0">
        <span className="font-body text-xs text-foreground truncate">{primary}</span>
        {chip}
      </div>
      {secondary && <span className="font-body text-xxs text-[#687182] truncate">{secondary}</span>}
    </div>
  );
}

// Currency cell — amount + currency code pair, always align="end"
export function CurrencyCell({ amount, currency = "EGP" }: { amount: string; currency?: string }) {
  return (
    <span className="inline-flex items-baseline gap-[2px]">
      <span className="text-[13px] text-foreground">{amount}</span>
      <span className="text-xs text-[#687182]">{currency}</span>
    </span>
  );
}

// Linked cells — record ids semibold, dates regular, both in --link (Ocean 80)
export function IdCell({ children }: { children: React.ReactNode }) {
  return <span className="font-semibold text-ocean-80">{children}</span>;
}
export function DateCell({ children }: { children: React.ReactNode }) {
  return <span className="text-ocean-80">{children}</span>;
}

// StatusChip — canonical status component (Figma "Status" 151:8361/8363, "StatusXLarge" in POS)
// NOT the pill Badge: tiny rectangle, radius 2, Noto Sans REGULAR (not bold) uppercase.
export function StatusChip({ tone, size = "sm", children }: {
  tone: "positive" | "negative" | "neutral" | "warning";
  size?: "sm" | "xl";        // sm = tables (18px tall) · xl = POS / large surfaces
  children: React.ReactNode;
}) {
  const tones = {
    positive: "bg-[#D1FFD3] text-[#1A541D]",   // Success 08 / Success 01 — active, approved, paid
    negative: "bg-[#FFEAED] text-destructive", // Error 09 / Error Base — inactive, rejected, failed
    neutral:  "bg-muted text-cadet-base",
    warning:  "bg-mango-10 text-warning-foreground",
  };
  return (
    <span
      className={cn(
        "inline-flex items-center justify-center font-body font-normal uppercase leading-[1.2]",
        size === "sm" ? "px-1 py-0.5 rounded-sm text-xxs" : "px-2.5 py-[5px] rounded-[5px] text-base",
        tones[tone],
      )}
    >
      {children}
    </span>
  );
}

// Row actions cell — stop propagation so onRowClick doesn't fire
export function RowActions({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex justify-end gap-1" onClick={e => e.stopPropagation()}>
      {children}  {/* icon Buttons or a DropdownMenu trigger */}
    </div>
  );
}
```

## 2. EmptyState

Required whenever a table or list can be empty. Also reusable full-page (e.g., first-run screens).

```tsx
// components/ui/EmptyState.tsx
export function EmptyState({
  icon: Icon = InboxIcon,
  title,
  description,
  action,
}: {
  icon?: React.ComponentType<{ className?: string }>;
  title: string;
  description?: string;
  action?: React.ReactNode;   // <Button>
}) {
  return (
    <div className="flex flex-col items-center justify-center text-center gap-3 py-14 px-6">
      <div className="w-12 h-12 rounded-full bg-muted flex items-center justify-center">
        <Icon className="w-6 h-6 text-muted-foreground" />
      </div>
      <div className="flex flex-col gap-1">
        <p className="font-body text-base font-semibold text-foreground">{title}</p>
        {description && <p className="font-body text-xs text-muted-foreground max-w-sm">{description}</p>}
      </div>
      {action}
    </div>
  );
}
```

```tsx
<EmptyState
  icon={ArrowsUpDownIcon}
  title="No transactions yet"
  description="Transactions appear here as soon as you accept your first payment."
  action={<Button size="sm"><PlusIcon className="w-4 h-4" />Create payment link</Button>}
/>
```

## 3. TablePagination

Canonical spec — Figma "Table pagination" node `151:8600`. A compact 44px bar: **"X-Y of Z" · "Rows per page: N ▾" · prev/next icon buttons with a "page/pages" fraction between them.** There are NO numbered page buttons in the design system — do not add them. The bar is compact enough to work unchanged at every viewport width.

```
Bar     : h-11 (44px) · px-5 py-3 · gap-5 · bg #FDFDFD (Neutral 11) · backdrop-blur 4px
          border-t · rounded-b (matches table container radius)
Text    : Inter Medium 12px · #687182 (Gray/500) · tracking 0.36px
Count   : "1-10 of 97" — start side, flex-1
Per page: "Rows per page: 10 ▾" — borderless trigger, inline 16px chevron
Buttons : icon-only, 16px chevron icon, px-1 py-0.5, radius 6
          enabled  = bg white + shadow 0 1px 1px rgba(0,0,0,.1),
                     0 0 0 1px rgba(70,79,96,.16), 0 2px 5px rgba(89,96,120,.1)
          disabled = bg #F7F9FC (Gray/0) + ring 0 0 0 1px rgba(70,79,96,.24)
Fraction: "1/10" between the buttons — current page #171C26, "/total" #687182
```

```tsx
// components/ui/TablePagination.tsx
import { ChevronLeftIcon, ChevronRightIcon, ChevronDownIcon } from "@heroicons/react/16/solid";
import { cn } from "@/lib/utils";

export interface TablePaginationProps {
  page: number;                 // 1-based
  total: number;                // total row count
  perPage: number;
  perPageOptions?: number[];    // default [10, 20, 50]
  onPageChange: (page: number) => void;
  onPerPageChange?: (perPage: number) => void;
}

const CAPTION = "font-[Inter,sans-serif] text-[12px] font-medium leading-[18px] tracking-[0.36px] text-[#687182]";

export function TablePagination({
  page, total, perPage, perPageOptions = [10, 20, 50], onPageChange, onPerPageChange,
}: TablePaginationProps) {
  const pageCount = Math.max(1, Math.ceil(total / perPage));
  const from = total === 0 ? 0 : (page - 1) * perPage + 1;
  const to = Math.min(page * perPage, total);

  return (
    <div className="flex items-center gap-5 px-5 py-3 border-t border-border-subtle bg-[#FDFDFD] backdrop-blur-[4px] rounded-b">
      {/* Count — start side, grows */}
      <span className={cn(CAPTION, "flex-1 min-w-0 whitespace-nowrap")}>
        {from}-{to} of {total}
      </span>

      {/* Rows per page — borderless trigger with inline chevron */}
      {onPerPageChange && (
        <label className={cn(CAPTION, "flex items-center gap-1 whitespace-nowrap cursor-pointer")}>
          Rows per page:
          <span className="relative flex items-center gap-0.5">
            <select
              value={perPage}
              onChange={e => { onPerPageChange(Number(e.target.value)); onPageChange(1); }}
              className="appearance-none bg-transparent border-0 pe-4 font-[inherit] text-[inherit] tracking-[inherit] text-[#687182] cursor-pointer focus:outline-none"
            >
              {perPageOptions.map(n => <option key={n} value={n}>{n}</option>)}
            </select>
            <ChevronDownIcon className="w-4 h-4 absolute end-0 pointer-events-none text-[#687182]" />
          </span>
        </label>
      )}

      {/* Prev · fraction · next */}
      <div className="flex items-center gap-2.5">
        <PagerButton aria-label="Previous page" disabled={page <= 1} onClick={() => onPageChange(page - 1)}>
          <ChevronLeftIcon className="w-4 h-4 rtl:rotate-180" />
        </PagerButton>
        <span className={CAPTION}>
          <span className="text-[#171C26]">{page}</span>/{pageCount}
        </span>
        <PagerButton aria-label="Next page" disabled={page >= pageCount} onClick={() => onPageChange(page + 1)}>
          <ChevronRightIcon className="w-4 h-4 rtl:rotate-180" />
        </PagerButton>
      </div>
    </div>
  );
}

// Icon-only pager button — enabled: white + layered shadow · disabled: Gray/0 + flat ring
function PagerButton({ className, ...props }: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  return (
    <button
      className={cn(
        "flex items-center justify-center px-1 py-0.5 rounded-md text-[#464F60] transition-shadow",
        "bg-white shadow-[0px_1px_1px_0px_rgba(0,0,0,0.1),0px_0px_0px_1px_rgba(70,79,96,0.16),0px_2px_5px_0px_rgba(89,96,120,0.1)]",
        "disabled:bg-[#F7F9FC] disabled:shadow-[0px_0px_0px_1px_rgba(70,79,96,0.24)] disabled:cursor-not-allowed disabled:text-[#868FA0]",
        className
      )}
      {...props}
    />
  );
}
```

## 4. Wiring it together (list page)

```tsx
const [page, setPage] = useState(1);
const [perPage, setPerPage] = useState(20);
const [sort, setSort] = useState<SortState>(null);

const sorted = useMemo(() => sortRows(rows, sort), [rows, sort]);
const pageRows = sorted.slice((page - 1) * perPage, page * perPage);

<DataTable
  toolbar={
    <TableToolbar title="All Transactions">   {/* search lives in the topbar — not here */}
      <ToolbarLink icon={FunnelIcon} onClick={() => setFiltersOpen(o => !o)}>Filter</ToolbarLink>
      <ToolbarLink icon={ArrowDownTrayIcon}>Export</ToolbarLink>
    </TableToolbar>
  }
  columns={[
    { key: "reference", header: "Reference", sortable: true },
    { key: "customer",  header: "Customer" },
    { key: "status",    header: "Status",
      render: r => <StatusChip tone={r.status === "paid" ? "positive" : "negative"}>{r.status}</StatusChip> },
    { key: "date",      header: "Date", sortable: true },
    { key: "amount",    header: "Amount", align: "end", sortable: true,
      render: r => <CurrencyCell amount={r.amount} /> },
  ]}
  rows={pageRows}
  loading={isLoading}
  sort={sort}
  onSortChange={setSort}
  onRowClick={r => navigate(`/payments/${r.id}`)}
  emptyState={<EmptyState title="No transactions yet" description="…" />}
  pagination={{ page, total: rows.length, perPage, onPageChange: setPage, onPerPageChange: setPerPage }}
/>
```

Server-side data: pass `total` from the API response and fetch on `page`/`perPage`/`sort` change — the components are already controlled.

## 5. Responsive Strategy — pick ONE per table

| Situation | Strategy | How |
|-----------|----------|-----|
| Many columns, data-dense (reports, reconciliation) | **Horizontal scroll** (built-in default) | The `overflow-x-auto` wrapper + `min-w-[640px]` already handle it. Keep the first (identifying) column narrow. |
| Record-like rows (transactions, customers, payouts) | **Collapse to cards/list `<md`** — preferred | Render `DataTable` in `hidden md:block` and `RecordCard`s (preferred for entities — `lists.md` §1) or a `ListView` in `md:hidden` from the same data (below). But first ask: should this be a table at all? Customers and payment links are RecordCard lists at every width. |

**Column → list-row mapping recipe:** identifying column → `title` · secondary column → `subtitle` · amount → `trailing` value · status → trailing `StatusChip` · row click → item press.

```tsx
{/* ≥md: full table */}
<div className="hidden md:block">
  <DataTable columns={columns} rows={pageRows} pagination={paginationProps} … />
</div>

{/* <md: list from the same data (ListView/ListItem — see lists.md) */}
<div className="md:hidden">
  <ListView
    loading={isLoading}
    emptyState={<EmptyState title="No transactions yet" />}
    footer={<TablePagination {...paginationProps} />}
  >
    {pageRows.map(r => (
      <ListItem
        key={r.id}
        title={r.customer}
        subtitle={`${r.reference} · ${r.date}`}
        trailing={
          <div className="flex flex-col items-end gap-1">
            <CurrencyCell amount={r.amount} />
            <StatusChip tone={r.status === "paid" ? "positive" : "negative"}>{r.status}</StatusChip>
          </div>
        }
        onPress={() => navigate(`/payments/${r.id}`)}
      />
    ))}
  </ListView>
</div>
```

## 6. Table Checklist

- [ ] Pagination present (or an explicit reason the data set is bounded and small)
- [ ] Empty state with icon + message (+ CTA when the user can create the missing thing)
- [ ] Loading skeleton (never a blank white box or a lone spinner in a 500px void)
- [ ] `<md` strategy chosen: scroll or list collapse
- [ ] Amount columns `align: "end"` with `CurrencyCell`
- [ ] Status shown as `StatusChip` (radius 2, Noto Sans regular uppercase), not plain text or pill Badge
- [ ] Row click → detail page (plus `RowActions` only for secondary actions)
