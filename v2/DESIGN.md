# Dashboard Design System
**Home Assistant Tablet Dashboard — Malibu House**
Last updated: April 2026

Built with `custom:button-card`, `custom:layout-card`, and `custom:decluttering-card`. No Bubble Card, no Swipe Card. All content rendered as HTML inside button-card `custom_fields` to avoid shadow DOM theming conflicts.

---

## Layout

### Grid Structure
Every view uses the same root grid:

```
┌─────────────┬──────────────────────────────┐
│             │                              │
│   Sidebar   │      Content (2 cards)       │
│   210px     │         1fr                  │
│             │                              │
├─────────────┴──────────────────────────────┤
│              Footer / Pills                │
│                  160px                     │
└────────────────────────────────────────────┘
```

| Dimension | Value |
|-----------|-------|
| Sidebar width | `210px` |
| Footer height | `160px` |
| HA header offset | `56px` |
| Content card height | `calc(100vh - 56px - 160px - 144px)` |
| Content card top margin | `100px` |
| Content card side padding | `0 60px 0 20px` |
| Gap between content cards | `150px` |

### Why These Numbers
- `56px` = HA's top header bar (hidden in kiosk mode but still in DOM)
- `144px` = 100px top margin + 44px bottom clearance
- `150px` card gap = intentional breathing room between the two content cards

---

## Color Palette

| Token | Value | Usage |
|-------|-------|-------|
| Page background | `oklch(96% 0.008 60)` | View background (deepest layer) |
| Footer bar | `rgba(255, 255, 255, 0.4)` | Pills footer bar |
| Inactive tile | `oklch(91.5% 0.013 60)` | Off-state tile background |
| Content card | `rgba(255,255,255,0.2)` | White cards (semi-transparent for future bg images) |
| Nav text / labels | `oklch(20% 0.018 45)` | All UI text (espresso) |
| Muted subtext | `oklch(40% 0.015 48)` | Secondary text, off-state icons |
| Active blue | `#2B8FE0` | Active tiles, nav indicator, lights |
| Accent teal | `#1AABB8` | Temperature, indoor sensors |
| Orange | `#E8793A` | Weather icon, thermostat pill |
| Purple | `#9B59B6` | Scenes pill |
| Slate | `oklch(58% 0.012 50)` | System pill |

### Weather Gradient Map
Each condition gets a vertical gradient (`180deg`) on the weather pill circle:

| Condition | Gradient |
|-----------|----------|
| sunny | `#1a6bb5 → #74b9ff` (dark→light blue) |
| clear-night | `#0a0a2e → #2C3E7A` (deep navy) |
| partlycloudy | `#0984e3 → #74b9ff` |
| cloudy | `#636e72 → #b2bec3` |
| rainy | `#1a6bb5 → #4a9eed` |
| pouring | `#1e3799 → #3d7abf` |
| snowy | `#89b4cc → #d0e8f5` |
| windy | `#0984e3 → #00cec9` |
| fog | `#778ca3 → #b2bec3` |
| lightning | `#e17055 → #fdcb6e` |
| snowy-rainy | `#4a9eed → #d0e8f5` |
| hail | `#2d3436 → #636e72` |
| default | falls back to `sunny` |

---

## Typography

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Clock | `58pt` | `200` | `#7A8394` |
| Nav items | `24px` | `400` | `#7A8394` |
| Card title | `20px` | `400` | `#7A8394` |
| Tile label | `15px` | `600` | white or `#7A8394` |
| Tile subtext | `13px` | `400` | `rgba(255,255,255,0.8)` or `#B8BEC9` |
| Pill label | `18px` | `600` | `#7A8394` |
| Pill subtext | `13px` | `400` | `#B8BEC9` |
| Weather subtext | `13px` | `400` | `#B8BEC9` |

Font family: `Roboto, sans-serif` (system default in HA)

---

## Components

### Sidebar
A `vertical-stack` in grid column 1. Contains:
1. `spacer` (decluttering template) — 20px top breathing room
2. `clock_card` (decluttering template) — live JS clock, 12-hour format
3. 5× `nav_item` (decluttering template) — one per view, blue border on active
4. No weather widget — moved to footer pill

**Nav active indicator:** `border-left: 4px solid #2B8FE0` via `card_mod` on `ha-card`. Inactive uses `transparent`.

**Why `card_mod` for the border:** `styles.card border-left` doesn't override HA's theme. `ha-card { border-left: ... !important }` in card_mod does.

### Tile Grid
2×2 grid inside a content card. Layout: `grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr`.

The left tile typically spans both rows (`grid-row: 1/3`) for the primary device.

**Tile states:**
- **Active (colored):** Blue `#2B8FE0` or teal `#1AABB8`, white text, stronger shadow
- **Inactive:** `#E8EAED`, grey text `#7A8394`, lighter shadow

### Content Cards
All content cards are `custom:button-card` with:
- `position: relative` on the card
- `custom_fields.content` with `position: absolute; top:0; left:0; right:0; bottom:0`
- All HTML rendered inside `content` field

**Why absolute positioning on content:** button-card vertically centers its custom_fields by default. Absolute positioning breaks out of that centering entirely.

**Why not nested cards:** HA's card-in-card pattern has severe limitations — backgrounds get overridden by themes, margins accumulate, shadow DOM makes CSS targeting unreliable.

### Camera Cards
Camera tiles use a special pattern:
- `show_entity_picture: false` — prevents button-card's native image handling
- `cam` / `cam1` custom_field with JS template reading `entity_picture` attribute
- The `entity_picture` attribute already contains the authenticated proxy URL with token

```javascript
var e = states['camera.porch_snapshots_fluent'];
var url = e ? e.attributes.entity_picture : null;
// url = /api/camera_proxy/camera.xxx?token=...
```

**Why entity_picture:** HA camera proxy requires auth. The `entity_picture` state attribute provides the pre-authenticated URL. Direct `/api/camera_proxy/` calls return 403 without the token.

### Footer Pills
Single `custom:button-card` spanning `grid-column: 1/3`. All 6 pills (Scenes, Lights, System, Climate, Lock, Weather) generated in one JS template using a `pill()` helper function.

**Weather pill** is dynamically generated from `weather.forecast_home` state with night detection via `sun.sun`.

---

## Decluttering Templates

Five reusable templates defined at the top of the YAML (before `kiosk_mode`):

| Template | Variables | Purpose |
|----------|-----------|---------|
| `sidebar_menu`| `[page]_border` | Full vertical-stack nav sidebar. Active page gets blue border. |
| `spacer` | none | 20px transparent gap before clock |
| `clock_card` | none | Live 12-hour clock, no date |
| `nav_item` | `name`, `path`, `border` | Single nav link with active indicator |
| `footer` | none | Full footer bar with all pills and weather |

**Why not `content_card` template:** Passing complex HTML as a decluttering-card variable breaks YAML parsing. Only simple scalar variables work reliably.

---

## Known Limitations & Workarounds

### button-card centering
button-card always vertically centers `custom_fields` content. Fix: set `position: relative` on the card and `position: absolute; top:0; left:0; right:0; bottom:0` on the content field.

### Theme interference
HA themes override `background`, `border-radius`, and `box-shadow` on `ha-card`. Fix: set these in `styles.card` which applies inline styles that win over theme CSS.

### Nav border-left
`styles.card border-left` doesn't override theme. Fix: use `card_mod: style: ha-card { border-left: 4px solid #2B8FE0 !important }`.

### Camera auth (403)
`/api/camera_proxy/` requires a token. Don't try to construct the URL manually. Read `states['camera.entity'].attributes.entity_picture` which already has the token embedded.

### picture-entity height
`picture-entity` ignores explicit height and expands based on image aspect ratio. Workaround: use `custom:button-card` with `cam` custom_field returning `<img>` via JS template.

### Decluttering-card HTML variables
Passing HTML strings as decluttering-card variables breaks YAML. Only use decluttering-card for templates with simple scalar variables (strings, colors, paths).

### No !include support
Dashboard is edited via HA's UI editor (paste), so filesystem-based `!include` is unavailable. Use `decluttering-card` for reuse instead.
