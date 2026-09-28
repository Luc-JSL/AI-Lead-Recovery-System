# Returnline HQ design system

Every new screen or component in `dashboard/` starts here. Before writing any color, spacing, radius or type value, use a token below. If something you need isn't covered, flag it and ask; don't invent a value.

The tokens live as CSS custom properties at the top of `returnline-hq.html`.

## Principles
- **Dark-first.** Deep navy is the default for every viewer. Light applies only when the viewer explicitly picks light (`data-theme="light"`).
- **One accent, reserved for action.** `--accent` marks primary buttons, the selected nav item, selected chips, focus rings and the next milestone. Never use it on chart data or status.
- **Inter for everything.** Numbers that line up use `font-variant-numeric: tabular-nums` (`.num`).
- **Dense.** Table rows 36px, controls 32px, 13px body text.
- **State is shown in form.** Status pills carry a dot and a word, never color alone.

## Tokens

### Color (dark / light)
| Token | Dark | Light | Use |
|---|---|---|---|
| `--bg-000` | #060b16 | #eef1f6 | Nav rail, segmented-control track |
| `--bg-100` | #0a1224 | #f5f7fb | Page |
| `--bg-200` | #0f1a31 | #ffffff | Cards, drawer, modal |
| `--bg-300` | #15223d | #f0f3f8 | Table header, row hover |
| `--bg-400` | #1c2c4d | #e3e9f2 | Selected nav item or tab, meter track |
| `--line-100` | #1c2a45 | #e1e6ef | Hairlines, card borders |
| `--line-200` | #5d72a0 | #76839c | Control borders (≥3:1) |
| `--ink-100` | #e6ecf7 | #0d1629 | Primary text |
| `--ink-200` | #a7b3cb | #46536d | Secondary text |
| `--ink-300` | #8190ad | #5f6c86 | Muted text, labels (only on bg-100/200/300) |
| `--accent` / `--on-accent` | #3ddcff / #04121c | #0070a8 / #ffffff | Primary action |
| `--good` `--warn` `--crit` `--info` | #3ecf8e #f2b441 #ff6e6e #8fb3ff | #177a4c #8f5a00 #c42b2b #2352a8 | Status pills and deltas only. Each has a `-soft` fill |
| `--series-1..3` | #3987e5 #d95926 #199e70 | #2a78d6 #eb6834 #1baf7a | Chart series, in this fixed order. Validated for colorblind separation |

All text pairs meet 4.5:1 in both themes on the grounds listed.

### Type (Inter)
| Size | Use |
|---|---|
| 11px / 600 / uppercase / .06em tracking | Labels, table headers, `.label` |
| 12px | Captions, secondary cells, chips, pills |
| 13px / 18px line height | Body and table cells |
| 14px / 600 | Card titles |
| 18px / 600 | Page title, drawer title |
| 28px / 600 | KPI values |

### Spacing, radius, sizes
- **Spacing:** `--sp-1` 4 · `--sp-2` 8 · `--sp-3` 12 · `--sp-4` 16 · `--sp-5` 20 · `--sp-6` 24 · `--sp-8` 32 · `--sp-10` 40.
- **Radius:** `--r-sm` 4 (skeletons, slots, chart bar ends) · `--r-md` 6 (cards, controls) · `--r-lg` 10 (modal) · `--r-pill` (pills, chips).
- **Sizes:** `--row` 36 · `--control` 32 · `--rail` 216 · `--drawer` 440 · `--modal` 420.

## Components
| Component | Class | Rules |
|---|---|---|
| Card | `.card` + `.card-h` / `.card-cap` / `.card-b` | One border, no shadow. The title is 14px. Actions go on the right of the header as `.btn.sm.ghost` |
| KPI tile | `.card.kpi` | Label, status pill, value, then an optional delta, an optional meter and a footer. A delta only shows when a prior period exists; otherwise the text says so |
| Data table | `.tbl` inside `.tbl-wrap` | Sticky-style header on `--bg-300`. Numbers right-aligned. Clickable rows get `.click`, `tabIndex=0` and open the drawer |
| Tabs | `.tabs` | Segmented control, used for time range and sub-views. The primary sections use the nav rail |
| Status pill | `.pill` + `good` / `warn` / `crit` / `info` / neutral | Always a dot and a word. `live` pulses the dot, only for work in progress |
| Button | `.btn`, `.primary`, `.ghost`, `.danger`, `.sm` | At most one `.primary` per view or overlay |
| Chips | `.chipset` | Multi-select filters |
| Empty state | `.empty` | A title that says what's missing, one sentence on how to fill it, and an optional single action |
| Skeleton | `.skel` | Shaped like the content it replaces |

## Drawer vs. modal
- **Use a drawer** to inspect or edit one record without losing your place: a shop's details, its history, logging a stage. It opens from the right and the page stays visible behind the scrim. Esc or Close dismisses it.
- **Use a modal** only to confirm a consequential or irreversible choice that needs a decision now: marking a shop as a paying client (it changes revenue) or as not interested. Keep it short: a question as the title, one sentence of consequence, then Cancel and the action. The action button is `.primary`, or `.danger` when the choice removes something.
- **Never** put a form longer than about 3 fields in a modal. Never open a modal from a modal. A drawer may open a confirming modal on top of itself.

## Empty and loading states
- **Loading:** skeletons in the exact shape of the content (KPI tiles, table rows, list lines). Charts wait for data; while data refreshes, the old render stays at reduced opacity. Never show a spinner-only page.
- **Empty:** a dashed `.empty` box inside the card that would hold the content, never a blank card. Say what's missing and how to get it ("No activity in the last 30 days. The trend starts with the first walk-in you log."), with one action when there's an obvious next step.
- **Offline or failed:** a `.notice` banner at the top, plus the affected component's empty state explaining what didn't load.
