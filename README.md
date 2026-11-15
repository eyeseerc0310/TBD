# STILL

A calm, local-first personal workspace — a place to capture, organise and revisit
thoughts, tasks, events and ideas.

Home has three columns: a clock / calendar card (swipe or use the arrows to switch
views) above a Connections card (your most recent note-to-note links), a task manager with weekly progress, colours and
drag-to-reorder, and a Notes card with a Title field and body, and the three most recently touched
notes pinned below (tap one to edit it in place). The clock view includes a
stopwatch / countdown timer; when the card is squashed vertically (the face under
about 190px tall; analog again from 210px), the clock crossfades to a large digital time and back; a narrow card
keeps the analog face, which then uses the card's full width. Drag the pill handles in the gaps between the cards to
resize them (double-tap one to reset it; Settings → Appearance → Reset card sizes
resets all). Around it:
universal Quick Capture, themeable settings, and a versioned backup system. The Connections
page shows how notes and pins link together, as a graph or a list. Files and Pin Board
are in place as workspaces ready to grow.

## Running it

STILL is a single file: `index.html`. No build step, no dependencies, no account.

- **Locally:** open `index.html` in any modern browser.
- **GitHub Pages:** publish the repository root; the page works as-is.
- **Offline:** once loaded, everything runs without a connection.

All data stays in your browser (`localStorage`). Nothing is sent anywhere, except
what you type into the top-bar search, which is sent to Wikipedia's public API to
find articles (article pages load from Wikipedia too). The only other network
request is the web font for the chosen typeface (Inter, JetBrains Mono or
Playfair Display, from Google Fonts); choose **SF Pro / System** to avoid even that.
Without a connection the app falls back to local fonts.

## Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| `Ctrl/⌘ K` | Quick Capture (from any page) |
| `Ctrl/⌘ Shift K` | Focus the Wikipedia search bar |
| `Ctrl/⌘ B` / `I` / `U` | Bold / italic / underline (in a note) |
| `Ctrl/⌘ K` in a note | Link the highlighted text to a web address (Quick Capture everywhere else) |
| `[[` in a note | Link to another note |
| `Ctrl/⌘ S` | Download a backup |
| `Esc` | Close dialogs, cancel edits |
| `N` | New task (Home) · Quick Capture (Notes) |
| `/` | Go to the search bar |
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
  links: ["id", …]          // ids of related items; for notes and pins, rebuilt from their content
}
```

Type-specific `meta`:

| Type | `meta` |
| --- | --- |
| `task` | `date` (`YYYY-MM-DD`), `done` (boolean), `order` (position within its day), `color` (`#rrggbb` or `null`) |
| `event` | `start`, `end` (`YYYY-MM-DD`), `startTime`, `endTime` (`HH:MM` or `null`), `allDay` |
| `capture` | `kind` (`"text"` today), `category`, `inbox`, `color`, `pinned`, `html` (formatted body; `content` keeps the plain text), `linkedAt` (when each note link first appeared). Captures are the saved notes shown on the Notes page and the Home notes card |
| `pin` | `attached` (ids the pin is attached to), `html` (optional body), `linkedAt`. The Pin Board UI is still to come; pins already take part in Connections |
| `connection` | Legacy saved links/contacts from the first Connections page: `target`, `kind`, `color`, `order`. Kept as they are; the Connections page offers to move them into Notes |
| `note` | `quick: true` marks the Quick Notes pad on Home (its text is `content`). "Add to Notes" moves the text into a `capture` (category `Note`, `source: "quick-notes"`) on the Notes page |

Dates and times are stored as local wall-clock values ("floating" time), so a 9:00
event stays at 9:00 across daylight-saving changes and time-zone moves.

Application state is `{ items, settings, ui, timer }`:

- `timer` — `mode` (`stopwatch` · `countdown`), `running`, `startedAt` (epoch ms), `elapsed` (ms before the current run), `duration` (countdown ms). Stored as timestamps, so it keeps time across refreshes.

- `settings`
  - Clock: `clock24`, `location`, `clockFace` (`minimal` · `classic` · `modern` · `digital`),
    `clockShape` (`circle` · `rounded` · `squircle` · `hexagon`), `showTime` (digital time under the face; off = larger face), `showSeconds`, `showDate`
  - Calendar: `weekStart` (`1` Monday, `0` Sunday), `highlightWeekends`
  - Appearance: `theme` (`graphite` · `midnight` · `forest` · `mocha` · `light` · `custom`),
    `customTheme` (`bg`, `panel`, `line`, `text`), `accent`, `font` (`inter` · `system` ·
    `mono` · `serif`), `transparency` (0–80 %), `blur` (0–40 px), `radius` (4–28 px),
    `density` (`comfortable` · `compact`), `navOrder` (order of the top-bar section icons),
    `cardSizes` (`null` for the defaults, or `{ cols: [a, b, c], split }`: the Home columns'
    shares of the width and the Clock/Calendar card's share of the left column, as fractions
    so they suit any screen)
- `ui` — `page`, `calMonth`, `calDay`, `taskDay`, `settingsSection` (last open Settings category), `homeView` (`clock` · `calendar`, the Home clock/calendar card)

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
- **Relationships:** every item has `links`; notes and pins already show up in Connections.

## Rich-text notes

The Notes card body is a rich-text editor (bold, italic, underline, strikethrough,
headings, bulleted / numbered / check lists, highlights, note links, Wikipedia links
and web links). Notes keep two copies of their body: `content` (plain text, used for
previews, search and the Notes page) and `meta.html` (the formatted version). HTML
is always cleaned against a short allow-list before it is shown or saved, so nothing
pasted, typed or restored from a backup can run. Notes from older versions have no
`meta.html` and load from their plain text.

### Formatting toolbar and links

The toolbar is one row of small icon buttons (text styles | headings and lists |
highlight | links) that scrolls sideways on narrow cards. The three link buttons work
on highlighted text and stay dimmed until something is highlighted:

- **Note link** — search your notes (or create one named after the highlighted
  words) and link to it. Tapping the link opens that note on the Notes page. A link
  whose note was deleted is greyed and marked "Missing note". Typing `[[` also opens
  the note picker and inserts a link.
- **W** — link to a Wikipedia article; opens in the in-app reader.
- **Web link** (`Ctrl/⌘ K`) — type an address (`https://` is added for you). The link
  shows a ↗ and opens in the device's browser, also from a home-screen app.
  Hover (or long-press on touch) shows the address with Edit / Remove.

Links keep the words they were made from: `<a data-note="id">`, `<a data-wiki="Title">`
or `<a href="https://…">`.

### Connections

Connections are the links between notes and pins: a note link connects two notes
both ways, and a pin connects with what it's attached to. Web and Wikipedia links are
never connections. Nothing is stored apart from the items: a note's `links` are read
from the `data-note` links in its text whenever it is saved or loaded (backups
included), so removing the last link to a note removes the connection. The Connections
page draws them as a graph (drag to pan, pinch or scroll to zoom, tap a node to open
it) or a list with "Links to" and "Linked from". Each open note shows its Backlinks.
