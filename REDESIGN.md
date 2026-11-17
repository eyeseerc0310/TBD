# STILL redesign

**Status: done.** Steps 1–4 are complete. Step 4 (what changed, measured before and after) is
at the end of this file. Branch: `redesign`, one small commit per step.

## How I audited it

- I opened every page on iPad landscape, iPhone and desktop, in all five themes, with sample data and with none.
- I forced each state: menus open, dialogs open, a toast, offline, empty.
- Colour contrast is measured from the colours the theme engine actually computes, laid over the real card glass.
- Tap targets were measured on the iPad.
- Font sizes, weights and corner radii were counted from what actually renders, not just what the stylesheet says.
- Every spacing, size, shadow and colour value in the stylesheet was counted.

## What already works (keep it)

- **The identity.** Dark glass, one accent, very little chrome. The clock gives the app a calm centre.
- **The theme engine.** Every theme is built from five base colours, so a fix in one place reaches every theme, including Custom.
- **Tokens exist** for spacing, type, corner radius and motion. The job is to finish the system, not replace it.
- **Keyboard focus** rings only show when you use the keyboard. Reduced motion is respected.

## 1. Inconsistencies

| Area | What I found |
|---|---|
| **Font sizes** | 13 different sizes on Home: 11, 12.5, 12.96, 13, 14, 14.48, 16, 20, 21, 28, 45, 46 and 97px. Two of them are rounding accidents.<br>12.5px is the size of almost half of all text, and 11px is used 25 times. The stylesheet hard-codes 11 more sizes outside the scale (10.5px, 0.7em, a clamp…). |
| **Weights** | Five weights (300, 400, 500, 600, 700). 700 appears in only 3 places. |
| **Letter spacing** | 9 one-off values, from −0.03em to 0.32em. |
| **Corner radius** | 6 different radii on screen (16, 9, 6.75, 6, 999px and 50%). Nine hard-coded radii (2–14px) ignore the *Corner roundness* setting, so some corners don't follow it.<br>Inner corners aren't concentric: a 9px button sits in a 16px card with 12px padding. |
| **Spacing** | 74 different paddings (30 of them one-offs like 3px, 5px 8px, 0 9px), 39 margins and 11 hard-coded gaps. Each card pads itself differently. |
| **Control heights** | 18, 22, 24, 26, 28, 30, 32, 34, 36 and 42px: 54 heights are hard-coded. The Notes page alone has icon buttons at 24, 26, 28 and 34px. |
| **Shadows** | 17 different shadows. They don't map to layers: a popover, a dialog, a toast and a card each invent their own. |
| **Blur** | One blur setting, plus four hard-coded blurs (3, 8, 10, 28px). |
| **Icons** | Stroke widths of 1.6, 1.8, 2.4, 2.5 and 3. Icon sizes from 12 to 24px.<br>In the top bar, the nav buttons are 28px and the buttons beside them are 34px. |
| **Buttons** | At least nine ways to draw something you can click: `.btn`, `.icon-btn`, `.link-btn`, `.tool`, `.chip`, and one-offs (the "Add a task" pill, the New note row, the agenda toggle, the view arrows).<br>Primary buttons are 28px on cards (Start, the + button) and 34px in dialogs. |
| **Disabled** | Four different treatments (opacity 0.38, 0.4 and 0.45, or nothing). |
| **Focus** | Four styles: an outline ring, a glow on inputs, a background colour only on menu items, and an inset ring on the graph. The top search turns its outline off. |
| **Card headers** | Each card does its own thing:<br>- Clock: a centred grey caption.<br>- Tasks: a 12.5px bold title with a counter.<br>- Notes: a 20px title with a tiny link.<br>- Connections: an icon only.<br>There is no shared header row. |
| **Layers** | 14 stacking (z-index) values from 1 to 100, with no order to them. 32 raw colour values sit outside the themes. |

## 2. Visual weaknesses

**Hierarchy.** Almost all text is between 11 and 12.5px. Only the clock leads, so task names, note titles, labels and metadata all read at the same volume. Page titles (26px) and card titles (12.5–20px) don't form a ladder.

**Contrast.** The faintest text colour (used for captions, times, placeholders and "1 of 2 done") fails AA in every theme. Light's red (used for Delete) fails badly. Control edges are nearly invisible.

| Theme | Faint text on a card | Red on a card | Input / control edge |
|---|---|---|---|
| Graphite | 3.82 ✗ | 6.48 ✓ | 1.21 ✗ |
| Midnight | 3.72 ✗ | 6.50 ✓ | 1.16 ✗ |
| Forest | 3.70 ✗ | 6.42 ✓ | 1.19 ✗ |
| Mocha | 3.68 ✗ | 6.34 ✓ | 1.19 ✗ |
| Light | 3.12 ✗ | 2.49 ✗ | 1.23 ✗ |

*Targets: 4.5 for text, 3.0 for edges that identify a control. Main text, secondary text, the accent and text on the accent already pass everywhere.*

**Density and clutter.**
- **The Home Notes card** stacks a title, a link, the editor, a 12-button toolbar and Recent in one column. On iPad the toolbar floats mid-card.
- **Settings** rows are cramped: 11–12.5px labels, with slider values at 11px.

**Dead space.**
- **Pin Board's** "Coming soon" sits alone in a panel the height of the screen.
- **Connections → List** leaves most of the screen empty.
- **The empty Tasks card** is a large void.

**Loudness in the wrong place.**
- **Finished tasks:** a coloured task that's done gets a full colour wash plus a strikethrough. In Light it's the loudest thing on screen, for something that's finished.
- **The timer:** 00:00:00 at display size competes with the clock's own time.

**Phone.** The clock card fills the whole first screen (about 600px), so tasks and notes start below the fold. The top bar and tab bar are heavy for the screen size.

## 3. Unpolished areas

**Touch targets (iPad).** There are no touch-specific sizes at all; nothing is styled for touch screens. Under 44px today:

| Element | Size today |
|---|---|
| Calendar days | 39 × 35 |
| Icon buttons | 34 and 28 |
| Notes toolbar tools | 26 |
| Week strip days | 28 wide |
| Task checkboxes | 18 drawn, 30 to tap |
| Drag handles | 16 × 30 |
| Folder chevrons | 22 |
| Graph / List switch | 26 tall |
| Text links ("Add to Notes", "View all") | 18 tall |
| Note rows | 32 |
| Folder rows | 36 |

**Hover-only controls.** Card buttons, the ⋯ on each row and the drag handles are invisible until you hover. On touch they show half-faded, which looks disabled.

**Feedback when you press.** There are 67 transitions but only 5 pressed (`:active`) styles, so most buttons don't react when you press them.

**Motion.**
- One easing curve for everything. A page change is a 4px fade.
- Reduced motion cuts every animation to 1ms, so even gentle fades vanish abruptly.

**Dialogs.**
- **Quick capture** works, but its tag chips, keyboard hint and buttons each use a different size.
- **New event** uses the browser's own date and time fields: their icons and font don't match the app, and they look different in Safari, Chrome and Firefox.

**Menus and popovers** (task ⋯, note ⋯, folder colour, graph tags). The structure is good. Item height, section labels and the Delete red vary, and menu items show focus with a background only.

**Toast.** "Task deleted · Undo" is small (11–12px), has no icon, and sits on top of the Tasks card.

**States.**
- **Offline:** shown only by a faint placeholder in the search bar, which is barely readable.
- **Loading:** only Wikipedia has a loading shimmer. Importing a backup shows nothing until it finishes.
- **Errors:** a red line under a field, or a red dot on a toast. The red fails AA in Light.
- **Empty states:** four different styles. Tasks has a check in a circle, Pins a "Coming soon" card, the Notes card a "No notes yet" line, and Connections a centred sentence.

**Performance.** 14 layers use a live background blur: the top bar, four cards, dialogs, popovers, the toast and the tab bar. That's the main cost on a Raspberry Pi, and today the only way to reduce it is the Blur slider.

## 4. Proposed direction: "Quiet glass"

**Fewer sizes, more air, clearer layers.** The clock and your own content lead, and the controls stay out of the way until you reach for them. It should feel like a well-made instrument: calm at rest, precise to the touch.

### Principles

**1. Three layers, one recipe each.**
- **Canvas:** the ambient background.
- **Surface:** cards and the top bar.
- **Floating:** popovers, dialogs and the toast.

Each layer has one fixed recipe: fill, 1px hairline, a soft top highlight and a shadow. Nothing else invents its own shadow or blur. Stacking order becomes five named levels.

**2. Seven type roles, nothing in between.**

| Role | Size and weight | Used for |
|---|---|---|
| Caption | 12 / 500 | The smallest text anywhere |
| Footnote | 13 / 400 | Times, counts, hints |
| Body | 15 / 400 | Task names, notes, rows |
| Headline | 15 / 600 | Card titles |
| Title | 17 / 600 | Dialog and section titles |
| Large title | 28 / 600 | Page titles |
| Display | light | Clock and timer numerals |

- Fixed line heights (1.2 for titles, 1.45 for text), with tighter letter spacing as sizes grow.
- Tabular figures for every time, date and count, so digits don't jiggle.
- Small per-font corrections, so Mono and Playfair sit at the same visual size as Inter.

**3. Spacing on a 4/8 grid.**
- Cards pad 20px (16 in Compact).
- Every card gets the same 44px header row: the title on the left, quiet metadata after it, and actions on the right.

**4. Corners from one dial.** Five radii (xs, sm, md, lg, pill), all driven by the *Corner roundness* setting. Inner corners are calculated to be concentric with the outer ones. No raw radii.

**5. One button family.**
- **Kinds:** primary (accent), secondary (tinted), plain (text), icon, destructive.
- **Sizes:** 28, 34 and 44px.
- **One set of states for all of them:** hover, press (a slight dip), the focus ring, disabled (40% and inert) and loading (a spinner that keeps the button's width).
- **Icons:** one stroke width, at three sizes (16, 20, 24).

**6. Colour with guarantees.** The theme engine already derives secondary and faint text from each theme. I'll change it to aim for a contrast ratio, not a fixed mix, so every theme meets AA, including any Custom theme you make:
- 4.5:1 for text.
- 3:1 for control edges.
- A proper red for Light.
- Matching success and warning colours.

Finished tasks go quiet: faded text with a small colour dot instead of a wash.

**7. Motion: calm, then precise.**
- **Durations:** four, at 120, 200, 300 and 450ms.
- **Easing:** a standard ease, plus a gentle spring for things that move under your finger (with a fallback for older browsers).
- **Where it moves:**
  - Everything tappable dips slightly when pressed.
  - Popovers grow from the button that opened them.
  - Dialogs rise.
  - Pages cross-fade.
- **Reduced motion:** fades only, with no movement.

**8. Glass you can read.**
- Live blur only on things that float over content: the top bar, popovers, dialogs and the toast.
- Cards use a lighter, cheaper glass.
- The ambient background gets a slightly richer, slower glow, still subtle.

**9. Touch first on iPad.**
- On touch screens every control gets a 44px tap area, even when it looks smaller.
- Row actions are always visible at full strength.
- Bigger calendar days, and a roomier Notes toolbar.

**10. One focus ring.** A 2px accent ring, 2px off the edge, on everything including inputs and menu items. It shows only for keyboard use.

### Screen by screen

- **Top bar:** one button size, with nav and actions aligned. The search shows an offline state you can actually read.
- **Clock card:** a quieter timer, with the Start and Reset buttons at a proper size. On phones the clock is smaller, so tasks show on the first screen.
- **Calendar:** bigger day cells, a clearer "today", and readable event dots.
- **Tasks:** 15px rows, a full-size checkbox, a quiet done state, and the ⋯ always reachable on touch.
- **Notes card:** a header row, with the editor and toolbar grouped so the toolbar sits under the text.
- **Connections card:** the same header row as the other cards.
- **Notes page:** one row height for folders and notes, with the sidebar and editor on the shared type scale.
- **Connections page:** list cards that fill the width, and graph controls in the shared button style.
- **Pins:** a compact empty state, not a full-height panel.
- **Settings:** roomier rows and clear section headers. Slider values at a readable size.
- **Dialogs:** the same header, body and footer structure for each one.
- **New event:** custom date and time fields in the app's style, following the 12/24h setting. The data saved is unchanged.
- **Toast:** larger, with an icon. It's placed so it doesn't cover what you just acted on.

### Performance mode (for the Raspberry Pi)

A new switch in **Settings → Appearance**, off by default. When on:
- Surfaces become solid instead of blurred.
- The background stops moving.
- Shadows are simpler, and transitions are shorter and fewer.

Every feature stays the same.

**On your data rule:** this adds one new optional setting (`performanceMode`, true or false). Nothing is renamed or removed. Older backups load unchanged, and a backup without it simply means off. Your Step 3 asks for this setting, so I'm treating it as allowed. Tell me if you'd rather it lived somewhere else.

## Plan for Steps 2–4

Each step is one small commit, so you can review or roll back any single one:

1. Design tokens: spacing, radius, layers, shadows, type and motion. The new tokens only, so nothing looks different yet.
2. Theme contrast: AA colours for every theme, plus success and warning.
3. Type scale applied everywhere.
4. Spacing and radii applied, and the one-off values removed.
5. Buttons and icons: one family.
6. Inputs and controls: fields, switches, segmented controls, checkboxes, sliders and chips.
7. Glass layers and the shared card header.
8. Overlays: dialogs, popovers, menus, the toast and tooltips.
9. Home cards, one by one.
10. The Notes, Connections, Pins and Settings pages.
11. Motion and press feedback.
12. Touch targets on iPad.
13. Performance mode.
14. Responsive passes for phone, iPad portrait and landscape, desktop and the Pi's screen.
15. Final check. I'll add before/after notes to this file.

**Testing after every commit:**
- The existing Playwright suites, about 1,100 checks.
- Screenshots in every theme.
- The contrast and tap-target measurements above, run again.
- Old backups (v1 files from earlier versions) loaded and round-tripped.

## Two decisions for you

1. **Bigger text.** Body text would go from 12.5px to 15px, and the smallest text from 11px to 12px. That's the biggest visible change, and I recommend it. Compact density would stay close to today's sizes for anyone who wants more on screen.
2. **Performance mode** adds one optional setting, as described above. OK?

---

## Step 4: what changed, before and after

Everything below was measured the same way as the audit, on the finished `redesign` branch.

### The commits

Each step is one commit, so any one can be reviewed or reverted on its own. Fix commits follow
the step they correct.

| # | Commit | What it does |
|---|---|---|
| 1 | Design tokens | The new tokens only; pixel-identical to before |
| 2 | Theme contrast | AA colours in every theme, Custom included |
| 3 | Type | Seven roles, 15px body, 12px floor |
| 4 | Spacing and radii | Every corner from the one dial; spacing on the 4/8 grid |
| 5 | Buttons and icons | One family, one set of states, one stroke |
| 6 | Fields and controls | Visible edges, one focus ring, iOS-like switches |
| 7 | Glass | One surface recipe, a richer ambient light, one header row |
| 8 | Overlays | One floating recipe, roomier dialogs, a toast with an icon |
| — | Fix | Recent's height follows the new row gap |
| 9 | Home cards | Soft rows, quiet finished tasks, a clearer calendar |
| 10 | Pages | Notes, Connections, Pins and Settings; one empty-state look |
| 11 | Event editor | The app's own date and time fields |
| 12 | Motion | Four durations, settling entrances, press feedback, calm reduced motion |
| — | Fix | Header rows start level with the week strip |
| 13 | Touch | Every control a 44px target on touch screens |
| 14 | Performance mode | Solid surfaces and lighter effects for slower computers |
| — | Fix | On touch screens the toolbar's tools share its width |
| 15 | Responsive | A smaller clock card on phones; every screen size checked |

### Contrast (on a card)

| Theme | Faint text | Secondary text | Red | Control edge |
|---|---|---|---|---|
| Graphite | 3.82 → **4.96** | 7.16 → 7.16 | 6.48 → 6.48 | 1.21 → **3.40** |
| Midnight | 3.72 → **5.03** | 7.06 → 7.06 | 6.50 → 6.50 | 1.16 → **3.46** |
| Forest | 3.70 → **4.95** | 7.01 → 7.01 | 6.42 → 6.42 | 1.19 → **3.50** |
| Mocha | 3.68 → **5.06** | 6.83 → 6.83 | 6.34 → 6.34 | 1.19 → **3.46** |
| Light | 3.12 → **4.79** | 5.77 → **6.14** | 2.49 → **4.71** | 1.23 → **3.17** |

- **Every combination passes.** All 6 themes × 7 accents × 3 glass transparencies meet AA. The weakest is 4.51:1, the readable accent on a Custom theme.
- **What was solved.** Faint text, secondary text, the readable accent and the red, green and amber reach at least 4.5:1. Control edges reach at least 3:1.
- **What stayed.** Colours that already passed are unchanged.

### Everything else, measured

| Area | Before | After |
|---|---|---|
| Text sizes on Home | 13, including 12.96 and 14.48 | The roles (12, 13, 15, 17) and the clock and timer numerals*, plus the timer's Start/Reset, which scale with the card between 13 and 15 |
| Most common text size | 12.5px (rows, menus, buttons) | 15px |
| Smallest text | 9.5px (badges), 10.5px (phone tabs) | 12px |
| Radii that ignored Corner roundness | 9 | 0 |
| Shadows | 17, one per component | 3 recipes |
| Hard-coded blurs | 4 | 0 (the dialog scrim is a token) |
| Icon sizes | 10 (11–28px), 5 stroke widths | 16 / 20 / 24, plus 14 in a line of text; one drawn weight |
| Disabled looks | 4 | 1 |
| Focus styles | 4 | 1 |
| iPad controls under 44px | 101 on Home | 0, except the week strip's seven days (27 × 64) |
| Pressed (`:active`) states | 5 | Every button, row, chip, swatch and day |
| Reduced motion | Everything cut to 1ms | Nothing moves; overlays still fade |
| Live blur layers | 14 | The same in normal mode; 0 in Performance mode |

\* The 14px Wikipedia "W" glyph in the toolbar is drawn as text but is an icon.

### Checked

- **1,092 checks pass** across the existing suites and the three new ones. The 46 that don't are all old checks that pin a size or a pixel position from an earlier design, so they change by design:
  - *test12* (37) and *test13* (7) compare against builds from earlier releases.
  - *test20* (1) checks the old Notes sidebar.
  - *test9* (1) pins Recent's old pixel position.
  - *test19* tests the three-column Notes page that was replaced earlier.
  - Two suites needed a test-only timing fix: they scrolled to the Settings and Windows controls before those sections finished opening. With that fix, the nav-icon drag and the Windows mini-map drag pass.
- **Three new suites:**
  - *test26*: date and time fields, 40 checks.
  - *test27*: Performance mode, 18 checks.
  - *test28*: the settings matrix and old backups, 12 checks.
- **Settings matrix.** Every theme with every accent, every font with both densities and 12/24h, in dark and Light, on Home and Notes: no errors, no sideways scrolling, no clipped cards, and the clock follows the setting.
- **Old backups.** Three real backups from earlier versions load with every item and setting, and save back as `still-backup` v1. Their files had no `performanceMode`, so they load with it off.
- **Screen sizes.** Phone, iPad portrait and landscape, the Raspberry Pi's 800×480, 1024×600 and 1280×720 screens, and 1440 and 1920 desktops: no sideways scrolling and no clipped cards.

### Where the result differs from the plan

- **Phone, first screen.** The clock card is smaller (about 62% of the screen, down from 600px), so the timer and the start of the next card show on the first screen. That next card is Connections, not Tasks. On phones the cards follow your layout's reading order, and changing that order would have changed behaviour.
- **Toast position.** It's larger, has an icon, and stays clear of the phone's tab bar, but it's still at the bottom centre. Any fixed spot covers something; this one usually covers the least.
- **Performance mode and the background.** The background never moved, so there was nothing to stop. Instead, Performance mode makes the background a plain gradient.
- **Week strip days.** They're as wide as the card allows (seven share it) and 64px tall. They can't reach 44px wide without a wider card.
- **Field edges in Light.** Meeting 3:1 makes field outlines a little firmer than Apple's own. That's the cost of the contrast promise.

### Not changed

- **The data format.** The only addition is `performanceMode`. Backups are still `still-backup` v1.
- **Behaviour and shortcuts.** Every feature, drag, gesture and shortcut works as before. The date and time fields keep the inputs' interface, so the event editor's rules are unchanged.
- **Static and offline.** Still one static file that works offline, with no build and no server.
