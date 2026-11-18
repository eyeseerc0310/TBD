# STILL

A calm, local-first personal workspace — a place to capture, organise and revisit
thoughts, tasks, events and ideas.

Home has three columns by default (rearrange them in Settings → Windows: drag the cards on a
mini-map of Home to swap them, stack two in a column, or give one a column of its own; or
pick a preset): a clock / calendar card (swipe, or use the ‹ › at its edges, which sit
on the divider line both views share, to switch views; the calendar's ‹ › and Today change
the month, and its content scrolls down to the Scheduled list) above a Connections card (the graph of your notes, the same one as on
the Connections page, with its own zoom), a task manager with weekly progress, colours and
drag-to-reorder, and a Notes card with a Title field and body, and the three most recently touched
notes pinned below (tap one to edit it in place). The clock view includes a
stopwatch / countdown timer; when the card is squashed vertically (the face under
about 170px tall; analog again from 185px), the clock crossfades to a large digital time and back; a narrow card
keeps the analog face, which then uses the card's full width. The card can't be made shorter than the digital
time needs to clear the timer. Drag the pill handles in the gaps between the cards to
resize them (double-tap one to reset it; Settings → Appearance → Reset card sizes
resets all). Around it:
universal Quick Capture, themeable settings (with a Performance mode for slower computers),
and a versioned backup system. The Notes
page has a sidebar and the same editor as Home (it moves there while the page is open). The
sidebar is folders: a search field (with a filter menu) with Archive and Trash buttons beside
it, then each folder as a rounded row that opens to list its notes ("Unsorted" first, for
notes in no folder), and "New folder" at the end. Drag folders, or the notes in them, into a
new order (a note into another folder too); folders take a colour from their ⋯ menu. The Connections page shows how notes and
pins link together, as a graph or a list. Pin Board is in place as a workspace ready to
grow.

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
| `↑` / `↓` on an event's date or time | A day, or five minutes, later / earlier |
| Arrow keys, `Home`/`End`, `Enter` in an open date picker | Move by day and week, pick the day (`Esc` closes the picker first, then the dialog) |

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
| `capture` | `kind` (`"text"` today), `category`, `inbox`, `color`, `pinned`, `html` (formatted body; `content` keeps the plain text), `linkedAt` (when each note link first appeared), `folder` (a folder id or `null`: Unsorted), `order` (its place in its folder once you've arranged it, or `null`: newest first), `archived`, `trashedAt` (when it went to Trash; notes are deleted 30 days later), `tag` (a connection tag id or `null`). Captures are the saved notes shown on the Notes page and the Home notes card |
| `folder` | `color` (or `null`), `order` (its place in the sidebar). Notes folders; Projects, Ideas, Reference and Personal are created once (older backups get them too) |
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
    `density` (`comfortable` · `compact`), `performanceMode` (`true`: solid surfaces and
    lighter effects, for slower computers like a Raspberry Pi; older backups don't have it
    and load with it off), `colourHistory` (the colour picker's last six colours) and `colourSaved`
    (`[{ hex, name }]`; older backups load with both empty), `navOrder` (order of the top-bar section icons),
    `notesSizes` (the Notes page's two columns, as fractions)
  - Windows: `homeLayout` (`null` for the default, or `{ cols: [{ cards, w, split }] }`: two or
    three columns of `time` · `conn` · `tasks` · `notes`, at most two to a column; `w` is a
    column's share of the width and `split` the top card's share of a stacked column, as
    fractions so they suit any screen; on tablets in portrait three columns fold into two, and
    on phones the cards follow the reading order), `notesSidebarRight`. Older backups'
    `cardSizes` (`{ cols: [a, b, c], split }`) are read into the default layout
  - Graph: `graphLinkDistance` (30–300), `graphRepulsion`, `graphCentre`, `graphCluster`,
    `graphFriction` (0–100 each), `graphNodeSize` (3–16 px), `graphLabelFade` (30–200, the
    zoom ×100 below which labels fade out), `graphTags` (connection tags:
    `[{ id, name, color }]`)
- `ui` — `page`, `calMonth`, `calDay`, `taskDay`, `settingsSection` (last open Settings category), `homeView` (`clock` · `calendar`, the Home clock/calendar card), `notesView` (`unsorted` · `folder:<id>`: the folder you're in, where a new note goes; `archive` · `trash`: those lists), `notesFilter` (`all` · `text` · `links` · `media` · `pinned` · `coloured` · `tasks`), `notesOpen` (which Notes folders are open), `foldersSeeded`
- `graph` — the Connections graph as it was left: `pos` (`{ id: [x, y] }`, with a third
  value `1` when the dot is pinned), and a camera for each place it's shown, `view` (the
  Connections page) and `card` (the Home card): `{ cx, cy, k }`, the point at the centre and
  the zoom, 0.2–4. Only dots without a saved position are placed by the simulation

Older data without the newer fields loads with defaults; tasks without an `order`
are numbered by creation time.

## Theming

Every colour, radius, blur and font is a CSS variable on `:root`. A small script in
`<head>` (`window.StillTheme`) turns the saved settings into those variables before
the page first paints, so a saved theme never flashes the default. The app calls the
same function whenever an appearance setting changes. Text on coloured elements
(accent buttons, the selected day, swatches) picks dark or light ink automatically
for contrast.

Colours you pick (a task's, event's, note's, folder's or tag's colour, highlights, the accent, a
Custom theme's colours) come from the palette or from the app's own colour picker, which the
rainbow swatch (or a colour field) opens: a glass panel by its button (a sheet along the bottom
on narrow screens) with the colour split into the original and the new one (tap the original to
go back; tap the new one, or ⌄, for the palette and your saved colours: long-press or right-click
one to rename or delete it), a readability badge, the eyedropper (where the browser has one),
Copy and Close; a saturation/brightness square and a hue slider; a HEX / RGB / HSL field (tap
its label to switch); the last six colours applied, and + to save the colour. What's being
coloured follows live; ×, Enter or a tap outside keeps it, Esc puts the original back. Arrow
keys move the handles (Shift for bigger steps). The history and saved colours are settings
(`colourHistory`, `colourSaved`), so they're kept and go into backups.

Default palette: Slate `#223843`, Mist `#EFF1F3`, Wine `#750D37`, Sage `#4D9078`,
Mustard `#D7C62C`, Clay `#D77A61`.

### Design system

All sizes, spacing, radii, layers, shadows, type and motion come from the tokens at the
top of the stylesheet (see `REDESIGN.md` for the reasoning):

- **Type**: seven roles (caption 12 · footnote 13 · body 15 · headline 15/600 · title 17 ·
  large title 28 · light display numerals); Compact density is 14/12/16/26. Tracking follows
  the size; Mono and Playfair get their own corrections (`data-font` on `<html>`).
- **Spacing** on a 4/8 grid; **radii** from the one Corner roundness dial (`--r-xs` … `--r-xl`,
  `--r-pop` for popovers, concentric inside them); controls 28 · 34 · 44px; icons 16 · 20 · 24
  (and 14 inside a line of text), drawn with one stroke weight.
- **Three layers**: the ambient background, surfaces (cards, the top bar: `--shadow-1`) and
  floating things (popovers, dialogs, toasts: `--shadow-2`), with named stacking levels.
- **Contrast**: the theme engine solves secondary and faint text, the readable accent and the
  red, green and amber to at least 4.5:1, and control edges (`--line-ctl`) to 3:1, on the page,
  cards, raised surfaces and fields, for every theme including Custom.
- **States**: one button family (primary, secondary, plain, destructive, icon) with hover,
  press, focus, disabled (40%) and loading; one focus ring (2px of the readable accent).
- **Touch** (`pointer: coarse`): controls grow a little and every one gets a 44px target.
- **Motion**: four durations (120/200/300/450ms), a settling ease-out for things that arrive
  and a gentle spring (CSS `linear()`, with a fallback); with reduced motion nothing moves
  and overlays only fade.
- **Performance mode** (Settings → Appearance): the engine makes surfaces solid (the colours
  they'd show over the background) and turns blur off; shadows are small, the background is
  a plain gradient and transitions are shorter. Every feature is the same.

## Backup format

**Save** downloads `dashboard-backup-YYYY-MM-DD.json`:

```json
{
  "format": "still-backup",
  "version": 1,
  "app": "STILL",
  "createdAt": "2026-11-14T08:00:00.000Z",
  "data": { "items": [], "settings": {}, "ui": {}, "graph": {} }
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
   quick notes, calendar, the event editor and its date and time fields, tasks, capture,
   settings, inbox, placeholder pages), navigation, keyboard, drag-and-drop, the render loop
   and initialisation.

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
page draws them as a graph or a list with "Links to" and "Linked from"; the Home
Connections card shows the same graph.

One model and one simulation serve both graphs (moving a dot in one moves it in the
other); each view is a canvas with its own camera. The Home card fits the graph when it
first shows and whenever the card is resized, names its notes when there's room and leaves
out labels that would collide, and has the Connections page's icon in its corner.

On the Connections page the graph fills the page and is drawn on a canvas (sharp at any pixel density), laid out
by a small force simulation: links pull, dots push apart, a gentle pull keeps it
centred, notes with the same connection tag drift together, and dots don't overlap.
Spacing tightens as the graph grows, so large graphs get denser instead of spreading
off-screen. It only runs while something moves and stops when it settles, when another
page is shown or when the app is in the background.

- Drag empty space to pan; pinch, scroll or use the buttons to zoom (0.2×–4×, toward the
  pointer). Labels fade out as you zoom out.
- Tap a dot to open its note. Drag a dot to move it (its links pull their notes along); a
  throw keeps it sliding until friction stops it. Hold a dot (500 ms) to pin it in place
  (a ring marks it); hold again to unpin.
- Keyboard: arrows step through the dots, Enter opens one, P pins it, + / − zoom.
- Connection tags (a note's ⋯ menu on the Notes page → Connection tag) colour a note's
  dot and group it with others of the same tag; untagged dots are grey.
- Settings → Connections has the graph's forces, sizes and label fade, and lists the tags.
