# STILL

A calm, local-first personal workspace — a place to capture, organise and revisit
thoughts, tasks, events and ideas.

The dashboard has a live clock with Quick Notes, an interactive calendar with events,
and a task manager with weekly progress, colours and drag-to-reorder. Around it:
universal Quick Capture, themeable settings, and a versioned backup system. Notes, Files, Connections and Pin Board
are in place as workspaces ready to grow.

## Running it

STILL is a single file: `index.html`. No build step, no dependencies, no account.

- **Locally:** open `index.html` in any modern browser.
- **GitHub Pages:** publish the repository root; the page works as-is.
- **Offline:** once loaded, everything runs without a connection.

All data stays in your browser (`localStorage`). Nothing is sent anywhere. The only
network request is the web font for the chosen typeface (Inter, JetBrains Mono or
Playfair Display, from Google Fonts); choose **SF Pro / System** to avoid even that.
Without a connection the app falls back to local fonts.

## Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| `Ctrl/⌘ K` | Quick Capture (from any page) |
| `Ctrl/⌘ S` | Download a backup |
| `Esc` | Close dialogs, cancel edits |
| `N` | New task (Home) · Quick Capture (Notes) |
| `/` | Search tasks (Home) |
| Arrow keys, `PgUp`/`PgDn`, `Home`/`End` | Move through the calendar |
| `Ctrl/⌘ Enter` | Save a capture |

Single-key shortcuts never fire while you are typing or while a dialog is open.

## Data model

All content lives in **one collection of items** with a shared shape, so search,
backups and relationships work the same way for every kind of content:

```js
{
  id: "uuid",
  type: "task" | "event" | "capture" | "note" | "file" | "connection" | "pin",
  createdAt: "ISO-8601",
  updatedAt: "ISO-8601",
  title: "",
  content: "",
  meta: { /* type-specific fields */ },
  links: ["id", …]          // ids of related items, any type
}
```

Type-specific `meta`:

| Type | `meta` |
| --- | --- |
| `task` | `date` (`YYYY-MM-DD`), `done` (boolean), `order` (position within its day), `color` (`#rrggbb` or `null`) |
| `event` | `start`, `end` (`YYYY-MM-DD`), `startTime`, `endTime` (`HH:MM` or `null`), `allDay` |
| `capture` | `kind` (`"text"` today), `category`, `inbox` |
| `note` | `quick: true` marks the Quick Notes pad on Home (its text is `content`) |

Dates and times are stored as local wall-clock values ("floating" time), so a 9:00
event stays at 9:00 across daylight-saving changes and time-zone moves.

Application state is `{ items, settings, ui }`:

- `settings`
  - Clock: `clock24`, `location`, `clockFace` (`minimal` · `classic` · `modern` · `digital`),
    `clockShape` (`circle` · `rounded` · `squircle` · `hexagon`), `showSeconds`, `showDate`
  - Calendar: `weekStart` (`1` Monday, `0` Sunday), `highlightWeekends`
  - Appearance: `theme` (`graphite` · `midnight` · `forest` · `mocha` · `light` · `custom`),
    `customTheme` (`bg`, `panel`, `line`, `text`), `accent`, `font` (`inter` · `system` ·
    `mono` · `serif`), `transparency` (0–80 %), `blur` (0–40 px), `radius` (4–28 px),
    `density` (`comfortable` · `compact`)
- `ui` — `page`, `calMonth`, `calDay`, `taskDay`, `settingsSection` (last open Settings category)

Older data without the newer fields loads with defaults; tasks without an `order`
are numbered by creation time.

## Theming

Every colour, radius, blur and font is a CSS variable on `:root`. A small script in
`<head>` (`window.StillTheme`) turns the saved settings into those variables before
the page first paints, so a saved theme never flashes the default. The app calls the
same function whenever an appearance setting changes. Text on coloured elements
(accent buttons, the selected day, swatches) picks dark or light ink automatically
for contrast.

Default palette: Slate `#223843`, Mist `#EFF1F3`, Wine `#750D37`, Sage `#4D9078`,
Mustard `#D7C62C`, Clay `#D77A61`.

## Backup format

**Save** downloads `dashboard-backup-YYYY-MM-DD.json`:

```json
{
  "format": "still-backup",
  "version": 1,
  "app": "STILL",
  "createdAt": "2026-11-14T08:00:00.000Z",
  "data": { "items": [], "settings": {}, "ui": {} }
}
```

The backup contains the complete application state — every item, all settings and
the current view — never just the visible page.

**Load** (button, or drag a file onto the window) goes through:
read → parse JSON → check format and version → validate structure → normalise
missing fields with defaults → show a summary and ask for confirmation → restore.

- Nothing changes until you confirm. Invalid, damaged or newer-version files are
  rejected and your data is left untouched.
- Unknown fields are preserved, so data from future versions isn't silently lost.
- Before a restore or a "Reset everything", the previous state is kept in
  `localStorage` under `still.state.before-restore` / `still.state.before-reset`.
- If stored data can't be read on start-up, the raw text is kept under
  `still.state.unreadable-<timestamp>` rather than discarded.

### Versioning

`version` is the schema version (`SCHEMA_VERSION` in the code). When the schema
changes, bump it and add a migration to `MIGRATIONS` in the Backup module:
`MIGRATIONS[n]` upgrades data from version `n` to `n + 1`. Older backups are
upgraded step by step on load; newer ones are refused rather than guessed at.

## Code structure

`index.html` contains, in order:

1. **CSS** — design tokens at the top (`:root`), then base styles, shared
   primitives (buttons, inputs, chips, dialogs), shell, pages, panels,
   responsive rules and motion.
2. **JavaScript** — one IIFE, in clearly marked sections: utilities, dates, model
   and normalisation, store (single source of truth), item/settings/UI actions,
   persistence (storage adapter), backup, search, toasts, dialogs, popover menus,
   colour picker, theme, clock face artwork, then one module per feature (clock,
   quick notes, calendar, event editor, tasks, capture, settings, inbox, placeholder
   pages), navigation, keyboard, drag-and-drop, the render loop and initialisation.

Data flows one way: an action changes the store → the store notifies → the page
redraws once per animation frame and persistence writes after a short debounce.

### Where it grows next

- **Storage:** replace `LocalAdapter` with an IndexedDB adapter before adding files
  or images (`localStorage` is limited to a few MB).
- **Capture kinds:** add entries to `KINDS` in the Capture module (links, images,
  files, screenshots, voice notes).
- **Global search:** `Search.query(q, { types })` already searches every item type.
- **Relationships:** every item has `links`; Connections can render them.
