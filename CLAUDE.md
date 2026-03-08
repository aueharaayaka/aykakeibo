# CLAUDE.md — aykakeibo

## Project Overview

**aykakeibo** (家計簿) is a Japanese household accounting progressive web app (PWA). It is a **zero-dependency, single-file application** — all HTML, CSS, and JavaScript lives in `index.html`. There is no build step, no package manager, and no frameworks.

The app is mobile-first, designed for iOS (installable as a home-screen PWA), with a pink color theme and Japanese UI text throughout.

---

## Repository Structure

```
aykakeibo/
├── index.html          # Entire application (HTML + CSS + JS, ~1924 lines)
├── icon.png            # App icon (180x180 PNG)
├── fonts/              # Self-hosted LINESeedJP font family
│   ├── LINESeedJP-Thin.ttf
│   ├── LINESeedJP-Regular.ttf
│   ├── LINESeedJP-Bold.ttf
│   └── LINESeedJP-ExtraBold.ttf
└── CLAUDE.md           # This file
```

All code is in `index.html`. There are no other source files to create or maintain.

---

## Technology Stack

- **Language**: Vanilla JavaScript (ES2015+), HTML5, CSS3
- **Frameworks/Libraries**: None
- **Build tools**: None
- **Package manager**: None
- **Data persistence**: `localStorage` (key: `kakeibo_state`)
- **Cloud sync**: Google Apps Script (Google Sheets backend)
- **Font**: LINESeedJP (self-hosted, 4 weights)

---

## index.html Structure

The file is organized in this order:

1. **`<head>`** — meta tags, font-face declarations, CSS custom properties
2. **`<style>`** — all CSS (~800 lines), organized with section comments like `/* ===== SCREENS ===== */`
3. **`<body>`** — all HTML DOM
4. **`<script>`** — all JavaScript (~1100 lines), organized with section comments like `// ===== STORAGE =====`

### JavaScript Section Map (by line number)

| Section | Lines | Description |
|---|---|---|
| State definition | ~760–802 | `state` object and `editTarget` |
| Storage | 808–819 | `saveData()`, `loadData()` |
| Init | 822–836 | Load data, initialize current month, `renderAll()` |
| Month key helpers | 839–846 | `monthKey()`, `getCurrentKey()`, `getCurrentData()` |
| Default expense logic | 849–865 | `getEffectiveDefault()`, `getAllExpenses()` |
| Month navigation | 868–884 | `changeMonth()` |
| Weekday counting | 887–904 | `isVacationDay()`, `countWeekdays()` |
| Recalculation | 907–932 | `recalc()` |
| Render orchestration | 933–946 | `renderAll()` |
| Month header | 947–967 | `renderMonthHeader()`, `goToToday()` |
| Category helpers | 968–973 | `tagClass()` |
| Expense list | 974–1015 | `renderExpenseList()` |
| Add/edit expenses | 1016–1148 | `addExpense()`, `deleteExpense()`, `editExpense*()`, `saveEdit()` |
| Total bar & filters | 1149–1194 | `renderTotalBar()`, filter sheet |
| Settings | 1195–1201 | `saveSettings()` |
| Vacation management | 1202–1479 | Vacation list, calendar picker (`vcal*` functions) |
| Default entries | 1411–1480 | `addDefaultEntry()`, `duplicateDefaultEntry()`, `deleteDefaultEntry()`, `renderDefaultList()` |
| Savings screen | 1481–1612 | `renderSavings()` |
| Screen navigation | 1613–1627 | `showScreen()` |
| Backup/Sync | 1628–1714 | `showBackupSheet()`, `syncUpload()`, `syncDownload()`, `exportData()`, `importData()` |
| Input handlers | 1715–1804 | Enter key handling, number formatting helpers |
| Month jump picker | 1805–1867 | `openMonthJumpPicker()`, `mjump*` functions |
| Toast notifications | 1868–1882 | `showToast()` |
| Pull-to-refresh | 1883+ | Touch gesture, cache-busting via URL reload |

---

## State Data Model

The entire app state is a single JavaScript object serialized to `localStorage`:

```javascript
state = {
  currentYear: 2026,         // number — currently displayed year
  currentMonth: 3,           // number — currently displayed month (1-12)
  totalFilter: '全体',       // string — active category filter
  data: {
    'YYYY-MM': {
      income: 0,             // number — monthly income
      expenses: [
        {
          id: string,        // unique ID (Date.now() based)
          date: 'YYYY-MM-DD',
          category: string,  // one of the 8 categories
          item: string,      // description
          amount: number     // can be negative (discounts)
        }
      ],
      overrides: {
        [defId]: {           // per-month override for a default entry
          deleted: boolean,  // true = soft-deleted for this month
          category: string,
          item: string,
          amount: number
        }
      }
    }
  },
  settings: {
    dailyFood: 0,            // number — daily food budget
    goal: 0,                 // number — monthly savings goal
    defaults: [
      {
        defId: string,       // unique ID (e.g. 'def_1234567890')
        category: string,
        item: string,
        amount: number
      }
    ],
    vacations: [
      { from: 'YYYY-MM-DD', to: 'YYYY-MM-DD' }
    ]
  }
}
```

**localStorage key**: `kakeibo_state`

---

## Expense Categories

There are exactly 8 categories. Do not add or remove them without updating `tagClass()`, all render functions, and the filter sheet:

| Category (Japanese) | Meaning | CSS tag class |
|---|---|---|
| 食費 | Groceries/food | `tag-food` |
| 外食 | Eating out | `tag-outside` |
| 交通費 | Transportation | `tag-transport` |
| 貯金 | Savings (inflow) | `tag-savings` |
| JO1 | JO1 savings (inflow) | `tag-jo1` |
| その他 | Other | `tag-other` |
| 貯金使用 | Savings usage (withdrawal) | `tag-savings-use` |
| JO1使用 | JO1 usage (withdrawal) | `tag-jo1-use` |

`貯金使用` and `JO1使用` are excluded from cash expense totals in `recalc()`.

---

## CSS Design System

All colors and radii are CSS custom properties defined in `:root`:

```css
--pink-bg: #fde8ef      /* page background */
--pink-light: #fdf0f4   /* card background */
--pink-main: #f0518a    /* primary accent */
--pink-dark: #e8256e    /* darker accent */
--pink-border: #f9c5d8  /* borders */
--pink-mid: #f7a8c4     /* muted/inactive */
--text-dark: #d41b6a    /* dark pink text, headers */
--text-body: #333       /* body text */
--white: #ffffff
--shadow: 0 2px 8px rgba(240,81,138,0.12)
--radius: 8px
--radius-sm: 8px
```

**Font**: `LINESeedJP` (self-hosted). Always specify `font-family: 'LINESeedJP', sans-serif`.

**Max width**: 430px, centered — this is a mobile-only layout.

**Safe area**: Use `env(safe-area-inset-top)` and `env(safe-area-inset-bottom)` for iOS notch/home bar support.

---

## Screen Architecture

Five screens, toggled by `showScreen(name)` which adds/removes the `.active` class:

| Screen ID | Nav label | Function |
|---|---|---|
| `screen-top` | ホーム | Monthly expense tracking |
| `screen-settings` | 設定 | Default entries, daily food budget, savings goal |
| `screen-savings` | 貯金 | Multi-year savings tracking table |
| `screen-vacation` | 休業日 | Vacation/holiday date range management |
| `screen-sync` | 同期 | Cloud sync (Google Sheets) and local backup |

---

## Key Conventions

### Coding Style
- **Camelcase** for all function and variable names
- **Inline event handlers** (`onclick="fn()"`) on DOM elements — do not add `addEventListener` calls in JS for UI events
- **No semicolons** omitted — semicolons are used consistently
- **Section comments**: Use `// ===== SECTION NAME =====` to delimit major sections in JS; `/* ===== SECTION ===== */` in CSS
- **Japanese comments** are common and expected — the codebase uses Japanese in comments, variable names, and all UI strings

### Input Handling
- Numbers use full-to-half-width conversion: `str.replace(/[０-９]/g, c => String.fromCharCode(c.charCodeAt(0) - 0xFEE0))`
- Comma-formatted numbers are stripped before parsing: `.replace(/,/g, '')`
- `formatNumberInput()`, `formatSettingInput()`, `formatGoalInput()`, `formatIncomeInput()` handle live formatting on `input` events
- IME composition is handled via `compositionstart`/`compositionend` — avoid triggering actions during Japanese input

### Default Entries vs. Monthly Expenses
- **Default entries** (`state.settings.defaults`) are recurring monthly expenses shown in every month
- They can be overridden per-month via `state.data['YYYY-MM'].overrides[defId]`
- An override with `deleted: true` hides the entry for that month only
- `getAllExpenses()` merges effective defaults + month-specific expenses
- `getEffectiveDefault(defEntry, overrides)` applies the override logic

### Month Range
- The app supports months from **2026-03 onwards** — earlier dates are clamped
- `changeMonth()` enforces this lower bound
- The upper bound is unlimited

### IDs
- Expense IDs: `String(Date.now())` at creation time
- Default entry IDs: `'def_' + Date.now()` or `'def_legacy_N'` for migrated old data

---

## Cloud Sync (Google Apps Script)

`syncUpload()` and `syncDownload()` communicate with a hardcoded Google Apps Script URL embedded in the JS. The URL is intentionally not stored in config — it is baked into the HTML.

- **Upload**: `POST` with JSON body `{ key: 'kakeibo', value: JSON.stringify(state) }`
- **Download**: `GET ?key=kakeibo`, returns `{ value: "..." }` JSON
- Uses `mode: 'no-cors'` for upload (fire-and-forget); standard fetch for download
- Both show a toast notification on success/failure

---

## Data Backup / Restore

- **Export**: `exportData()` creates a JSON blob download named `kakeibo_backup_YYYY-MM-DD.json`
- **Import**: `importData(e)` reads a file via `FileReader`, parses JSON, merges with current state

---

## Pull-to-Refresh

Implemented via touch events on the active screen. A downward swipe from the top triggers a cache-busting page reload (`location.href = location.pathname + '?v=' + Date.now()`). Data is preserved in `localStorage` before reload.

The `cache-version` meta tag and icon `?v=` query param are updated manually when deploying — they are not auto-incremented.

---

## Japan Holidays

`JAPAN_HOLIDAYS` is a `Set` of date strings (`'YYYY-MM-DD'`) hardcoded in the JS. It is used by `countWeekdays()` to exclude national holidays from the working-day count. Update this set when adding support for future years.

---

## Development Workflow

### Making Changes
1. Edit `index.html` directly — it is the only source file
2. Open `index.html` in a browser (or a local HTTP server for PWA features) to test
3. No compilation or build step is needed

### Testing
There are **no automated tests**. Verify changes manually in a mobile browser or browser DevTools mobile emulation (430px width, iOS UA string recommended).

### Git Workflow
- Branch naming: `claude/<task-slug>` for AI-assisted changes, `feature/<name>` or `fix/<name>` for manual work
- All branches merge to `master` via pull requests
- Commit messages are in Japanese or English — either is acceptable
- GPG signing is enabled (`commit.gpgsign=true` with SSH key)

### Deploying
The app is served as static files. After merging to `master`, deploy by copying the repository files to the web server. No build step required.

---

## Things to Avoid

- **Do not introduce dependencies** — no npm, no CDN imports, no frameworks
- **Do not split into multiple files** — the single-file architecture is intentional
- **Do not add a build step** — the app must remain directly openable as an HTML file
- **Do not change category names** — they are used as data keys in `localStorage`; changing them would corrupt existing user data
- **Do not remove the `cache-version` meta tag** — it is used by the pull-to-refresh mechanism
- **Do not use `document.querySelector` patterns that rely on class ordering** — prefer IDs for element access
- **Do not use `async/await` beyond the existing sync functions** — keep the codebase consistent with existing patterns
