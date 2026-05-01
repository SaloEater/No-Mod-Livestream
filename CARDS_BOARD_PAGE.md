# Cards Board Page — Planning

## Purpose

The **Cards Board Page** is a dedicated System page that displays all currently-available
valuable card photos from the active Series in a single rectangular area. It is designed
to be embedded as a Browser Source in OBS during a livestream. When a box is sold and its
valuable card is marked as sold, that card disappears from the display and the remaining
cards are reshuffled. The rectangle is always completely filled — no gaps, no empty space.

---

## What the Operator Sees

- A full-screen rectangular area covered edge-to-edge by card images.
- No UI controls, no scrollbars, no chrome — just the image grid.
- The images are arranged randomly and automatically resized so the entire rectangle is
  covered without any visible blank space.
- When a card is sold, it disappears and the remaining images are reshuffled and resized
  to fill the space again.

---

## What Triggers a Reshuffle

A reshuffle happens **only** when the set of displayed cards changes. Specifically:

| Event | Reshuffle? |
|---|---|
| A card is marked as sold and removed | Yes |
| Page is first loaded | Yes (initial random arrangement) |
| Page is refreshed | Yes (new random arrangement) |
| No cards change | No |

There is no time-based reshuffling. The layout is stable until a card is removed.

---

## Data Model

Each photo has an optional name (for internal accounting only — not displayed on the board)
and a sold status that defaults to false. A photo is available for display when it belongs
to the active Series and has not been marked sold.

There is no link between Photos and Spots. All photos in a Series appear on the board
until the operator marks them sold individually.

---

## System Requirements

The board needs two capabilities from the System:

- **Read**: Fetch all unsold photos from the Closed Series linked to the channel's active
  Livestream (resolved via Channel → Livestream → Breaks → Series). If no eligible photos
  exist, return an empty list.
- **Write**: Mark a photo as sold or restore it to unsold. Used by operator mode only.

Only **Closed** Series are shown — photos from a Series still being scanned are not
displayed even if it is linked to a break in the active Livestream. The Series–Break link
is set by the operator on the Spots Page before the stream starts.

The board detects sold cards automatically and updates the display without requiring a
manual refresh.

---

## Frontend Requirements

### Route

```
/channel/:channel/photos
```

This URL is what the operator pastes into OBS as a Browser Source. The System resolves
which Series to display automatically — the operator never needs to know a Series ID.

### Page Behaviour

On load, the page fetches available photos, shuffles them randomly, and renders the grid.
It then checks for changes in the background and updates the display whenever a card is
removed — no manual refresh is needed.

### No UI Controls Visible in OBS

The page must have no visible navigation, headers, or controls when used as an OBS source.
Operator controls are enabled separately via a query parameter:

```
/channel/abc123/photos?controls=true   ← operator mode (separate browser tab, not in OBS)
/channel/abc123/photos                 ← clean display only (pasted into OBS Browser Source)
```

The operator typically opens the controls URL in a regular browser tab on their screen
while OBS captures the clean URL in the background. Both point at the same channel; only
operator mode has interactive controls.

---

### Operator Mode (`?controls=true`)

Unlike the clean OBS display which shows only unsold cards, the operator page shows
**all** cards — both unsold and sold — in a single grid. Unsold cards come first, sold
cards follow, and the two groups are visually distinct.

#### 1. Full-Screen Card Preview on Hover

Hovering over any card shows a full-screen overlay of that image so the operator can
confirm it matches the physical card just pulled from a box.

- Overlay appears immediately on hover; no click required.
- Image fills the screen while preserving the card's proportions, on a dark backdrop.
- Moving the mouse off the card dismisses the overlay.
- Hovering does not change any state.

#### 2. Click a Card to Toggle Its Sold Status

Clicking any card toggles its sold status:

- Clicking an **unsold** card marks it as sold — it moves to the sold section of the
  grid and disappears from the OBS display.
- Clicking a **sold** card restores it — it moves back to the unsold section and
  reappears on the OBS display.

If the action fails, the card stays in its current state and an error is shown.

---

## Layout — Filling the Rectangle

Photos are arranged in a grid that covers the full viewport with no gaps. The number of
columns adjusts automatically based on how many cards are displayed and the viewport
dimensions, keeping cells as close to standard card proportions as possible. Each image
fills its cell completely, cropped to the centre if needed. If the last row has fewer
images than columns, those images are stretched to fill the remaining space.

The layout recalculates on initial load, whenever a card is removed, and whenever the
browser window is resized.

---

## OBS Integration

The operator adds the Cards Board Page as a **Browser Source** in OBS:

1. In OBS, add a new source → **Browser**.
2. Set URL to `http://<system-host>/channel/<channel>/photos`.
3. Set width/height to match the OBS canvas.
4. No custom CSS needed — the page is already full-bleed.

---

## Lifecycle Diagram

```
Operator sets up OBS Browser Source
      |
      v
Page loads → fetches available photos → shuffles → renders grid
      |
      v
Page checks for changes automatically  <------------------------+
      |                                                         |
      +-- No change → do nothing ------------------------------+
      |
      +-- Card was marked sold?
            |
            v
      Remove card, reshuffle, re-render grid ------------------+
      |
All cards sold → grid is empty → page shows blank screen
```

---

## What Is Out of Scope for This Step

- Authentication / access control for the board URL (the URL is treated as a secret
  share link for now).
- Animations or transitions between reshuffles (the grid simply re-renders; smooth
  animations can be added later).
- Multiple simultaneous Series on one board (one Series per board URL).
- Historical view of which cards have been sold.
