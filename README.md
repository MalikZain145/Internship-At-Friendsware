<div align="center">

# SPENDLY

### A clean, responsive personal finance tracker — no dependencies, no build step, no server

![Version](https://img.shields.io/badge/version-1.0.0-5b6af8?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-0f9d6e?style=flat-square)
![Zero Dependencies](https://img.shields.io/badge/dependencies-zero-d97706?style=flat-square)

</div>

---

## Table of Contents

1. [Overview](#overview)
2. [File Structure](#file-structure)
3. [Features](#features)
4. [How Data is Stored](#how-data-is-stored)
5. [Architecture & Code Design](#architecture--code-design)
6. [Responsive Design](#responsive-design)
7. [Getting Started](#getting-started)
8. [Customising](#customising)
9. [Browser Support](#browser-support)

---

## Overview

A fully client-side expense tracking application built with pure HTML5, CSS3, and Vanilla JavaScript. All state lives in the user's `localStorage` — no server, no database, no build pipeline, no `npm install` required.

It was built to demonstrate that a production-quality product does not require a framework. The entire application — multi-session management, live filtering, category breakdowns, print/export — is implemented in three clean files.

---

## File Structure

```
expense-tracker/
│
├── index.html     ← Application shell (markup only)
├── style.css      ← All styles, CSS variables, responsive breakpoints
└── app.js         ← All logic, state management, event handling, export
```

Each file has a single responsibility. `style.css` owns all visual decisions. `app.js` owns all behaviour. `index.html` owns the markup structure.

---

## Features

### Session Management

| Feature | Detail |
|---|---|
| Multiple sessions | Create and switch between independent expense sessions |
| Auto-naming | Sessions rename themselves based on active categories and creation date |
| Manual rename | Inline editable session name in the header |
| Delete session | Confirmation dialog before permanent deletion |
| Open in new window | Full-featured pop-out view for any session |
| Print session | Formatted printable report with category breakdown |

### Expense Management

| Feature | Detail |
|---|---|
| Add expense | Title, amount (PKR), and category |
| Inline validation | Field-level error messages on submit; clear the moment the field is corrected |
| Edit expense | In-place editing with a visual edit-mode indicator on the form card |
| Delete expense | Per-row delete with confirmation dialog |
| Category filter | Dropdown filters visible rows; totals reflect only the filtered view |
| Sort by amount | Toggle ascending or descending sort on the amount column |
| PKR formatting | All amounts formatted as `PKR 1,500` using `toLocaleString('en-PK')` |
| Empty state | Styled empty state card when no expenses exist or a filter returns zero results |

### Statistics & Breakdown

- **Grand total** — live-updating total for the active session
- **Filtered total** — total reflecting only the currently visible rows
- **Category breakdown panel** — proportional bar chart per category with amounts and counts

### UX

- Toast notifications for add, update, and delete actions
- Keyboard shortcuts: `Ctrl/Cmd + Enter` to submit the form, `Escape` to cancel edit mode
- Smooth row animations on add
- Edit mode highlights the active row and applies a warning border to the form

---

## How Data is Stored

The browser's built-in `localStorage` API is used. No data ever leaves the user's device.

### Storage Key

| Key | Type | Contents |
|---|---|---|
| `et_sessions_v2` | JSON string | Array of all session objects |

### Session Object Schema

```json
{
  "id": "ses_1717490000000",
  "name": "Food & Transport · Jun 4",
  "createdAt": 1717490000000,
  "nextId": 5,
  "expenses": [
    {
      "id": 1,
      "title": "Grocery run",
      "amount": 1500,
      "category": "Food"
    }
  ]
}
```

Sessions are stored as an array under a single key — the entire array is written on every mutation. This keeps the read/write logic simple and ensures consistency.

### Persistence Guarantees

- Adding or editing an expense → saved immediately before re-render
- Deleting an expense or session → saved immediately before re-render
- Renaming a session → saved on every keystroke
- Page refresh → full state restored on `init`

The load function wraps `JSON.parse` in a `try/catch` so the app never crashes on malformed storage data — it falls back to an empty sessions array.

---

## Architecture & Code Design

### Data Flow

The application follows a unidirectional data flow pattern — no library required:

```
User action
    ↓
Mutate in-memory array (sessions / expenses)
    ↓
Write to localStorage
    ↓
renderTable() re-derives the entire view from data
    ↓
DOM updated
```

`renderTable()` is the single source of truth for the table UI. Nothing is ever updated in isolation — the view always derives from the data.

### Event Delegation

A single listener on `<tbody>` handles edit and delete for all rows. This avoids attaching and leaking individual listeners as rows are added and removed dynamically:

```javascript
tbody.addEventListener('click', e => {
  const editBtn = e.target.closest('.btn-edit');
  const delBtn  = e.target.closest('.btn-delete');
  if (editBtn) enterEditMode(exp);
  if (delBtn)  confirmDialog(...);
});
```

### XSS Prevention

All user-provided strings written to the DOM are HTML-escaped before insertion:

```javascript
function escHtml(s) {
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

This prevents a title like `<script>alert(1)</script>` from executing in the DOM.

### CSS Design Tokens

All colours, radii, shadows, and fonts are CSS custom properties in `:root`. The entire visual system can be adjusted from one location:

```css
:root {
  --accent:      #5b6af8;
  --accent-dark: #4452e0;
  --success:     #0f9d6e;
  --warning:     #d97706;
  --danger:      #e03131;
  --radius-lg:   14px;
  --mono:        'DM Mono', monospace;
  --sans:        'Syne', sans-serif;
}
```

### Print & Export

The `printSession` and `openInNewWindow` functions generate a complete, self-contained HTML document as a string — including inlined CSS — and write it to a new browser window. The print function then calls `window.print()` on the new window. No library, no server, no PDF API.

---

## Responsive Design

The layout adapts across three tiers:

### Desktop (> 900px)
- Persistent left sidebar with full session list
- Three-column expense form
- Full table with all columns visible

### Tablet (700px – 900px)
- Sidebar remains visible
- Expense form collapses to a two-column grid

### Mobile (< 700px)
- Sidebar hidden off-screen, accessed via a hamburger menu button in the top bar
- Sidebar slides in as a fixed overlay with a blurred backdrop
- Sidebar closes on backdrop tap, session selection, or `Escape` key
- Expense form becomes single-column
- Expense table scrolls horizontally inside its container — the page itself does not scroll horizontally
- Sort button text labels hidden on very small screens (icons remain)

The page itself never scrolls horizontally at any viewport width. The expense table is the only element that scrolls, and only within its own bounded container.

---

## Getting Started

Clone the repository and open `index.html` in any browser — no server, no install:

```bash
git clone https://github.com/your-username/expense-tracker.git
cd expense-tracker
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

Or serve locally over `http://` for a proper origin:

```bash
# Python (built-in, no install)
python3 -m http.server 8080

# Node
npx serve .

# Then open http://localhost:8080
```

---

## Customising

| What to change | Where |
|---|---|
| Colours, fonts, spacing | CSS custom properties in `:root` in `style.css` |
| Expense categories | `<option>` elements in `index.html` and `cat-*` / `bar-*` CSS classes in `style.css` |
| Currency symbol | `fmt()` function in `app.js` |
| Amount format locale | `toLocaleString('en-PK')` call in `fmt()` |
| Print report styling | `printSession()` function in `app.js` |
| Session auto-naming logic | `autoRename()` function in `app.js` |

---

## Browser Support

| Browser | Supported |
|---|---|
| Chrome 80+ | ✓ |
| Firefox 75+ | ✓ |
| Safari 13.1+ | ✓ |
| Edge 80+ | ✓ |
| Samsung Internet 12+ | ✓ |
| Opera 67+ | ✓ |

Required APIs: `localStorage`, `CSS custom properties`, `CSS Grid`, `Array.prototype.reduce`, `Element.closest`. All available in every major browser since 2019.

---

<div align="center">

Built with care · Zero dependencies · Your data stays on your device

</div>
