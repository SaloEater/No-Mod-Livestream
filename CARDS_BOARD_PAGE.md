# Cards Board Page — Planning

## Purpose

The **Cards Board Page** is a dedicated System page that displays all currently-available
valuable card photos from a selected Series in a single rectangular area. It is designed
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

## Data Model Additions

The existing data model needs two additions to support this page:

### 1. Spot → `valuable_photo_id` (already implied in existing docs)

Each Spot can optionally reference a Photo from the Collection. This is the photo that will
appear on the Cards Board Page for that Spot.

```
Spot {
  id
  break_id
  label              (e.g. "Team name" or box number)
  buyer_id           (null until sold)
  valuable_photo_id  (null if this spot has no valuable card)
}
```

### 2. Derived state: "available" photos

A photo is considered **available** (displayable on the Cards Board) when:
- It is referenced by a Spot as `valuable_photo_id`, AND
- That Spot has no `buyer_id` (not yet sold).

There is no separate "sold" flag on the Photo itself. Availability is derived from
the Spot's sold state.

---

## Backend Requirements

### New Read Endpoint

```
GET /collections/:collectionId/board
```

Returns the list of photos that are currently available (valuable and not sold) for a
given Collection. Response shape:

```json
{
  "collectionId": "abc123",
  "photos": [
    { "id": "ph1", "url": "/photos/ph1.jpg" },
    { "id": "ph2", "url": "/photos/ph2.jpg" }
  ]
}
```

The backend computes availability by joining Spots with Photos — no extra storage field
is needed.

### Change Detection via Polling

The page detects changes to the available set by polling the board endpoint on a fixed
interval (every 5 seconds). On each poll:

1. Compare the returned photo IDs to the currently displayed set.
2. If the set is identical, do nothing — no reshuffle, no re-render.
3. If a photo is missing (sold), remove it, reshuffle, and re-render.

Polling every 5 seconds means the display updates within 5 seconds of a box being sold,
which is imperceptible during a livestream where boxes sell roughly once per minute.

### No New Write Endpoints

Marking a Spot as sold already happens on the Spots Page (assigning a buyer). The Cards
Board Page is read-only. No new write paths are required.

---

## Frontend Requirements

### Route

```
/board/:collectionId
```

This URL is what the operator pastes into OBS as a Browser Source.

### Page Behaviour

1. On load: fetch `GET /collections/:collectionId/board` to get the initial photo list.
2. Shuffle the list randomly (Fisher-Yates or equivalent).
3. Render all images in the layout described below.
4. Start a poll: repeat step 1 every 5 seconds.
5. On each poll response, compare photo IDs to the current set:
   - If unchanged: skip.
   - If a photo was removed: update the local list, re-shuffle, re-render.

### No UI Controls Visible in OBS

The page must have no visible navigation, headers, or controls. Any operator controls
(e.g. selecting a Collection) must be accessible only outside the OBS capture region, or
handled via query parameters:

```
/board/abc123?controls=true   ← show collection selector (for setup)
/board/abc123                 ← clean display only (for OBS)
```

---

## Layout Algorithm — Filling the Rectangle

The goal is to tile N images so they cover the full viewport rectangle with no gaps.

### Algorithm

Given N photos and a viewport of width W × height H:

1. Find the column count `C` that minimises wasted cells while keeping each cell's
   aspect ratio reasonably close to a card's aspect ratio (roughly 2.5 : 3.5 ≈ 5:7).
   - Try C from 1 to N; compute rows `R = ceil(N / C)`.
   - Score each (C, R) by how close `(W/C) / (H/R)` is to the card aspect ratio.
   - Choose the (C, R) with the best score.
2. Set a CSS Grid with `C` columns and `R` rows, each cell sized `W/C × H/R`.
3. Each `<img>` inside a cell uses `object-fit: cover` so it fills its cell completely
   with no letterboxing, cropping to the centre of the card image.
4. If `C × R > N` (there are empty cells at the end), the last row may have fewer than
   C images. Stretch those final images across the remaining columns using
   `grid-column: span 2` (or similar) so the bottom row is also fully covered.

### CSS Sketch

```css
.board {
  display: grid;
  width: 100vw;
  height: 100vh;
  /* column count and row count set dynamically via JS */
  grid-template-columns: repeat(var(--cols), 1fr);
  grid-template-rows: repeat(var(--rows), 1fr);
  overflow: hidden;
}

.board img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

### Re-layout Trigger

The layout is recalculated:
- On initial render.
- On every reshuffle (card removed).
- On browser window resize (relevant if OBS canvas size changes).

Because the column count depends on N and the viewport dimensions, removing one card
may change the optimal grid shape (e.g. from a 4×3 grid to a 4×2 grid when N drops
from 11 to 8).

---

## OBS Integration

The operator adds the Cards Board Page as a **Browser Source** in OBS:

1. In OBS, add a new source → **Browser**.
2. Set URL to `http://<system-host>/board/<collectionId>`.
3. Set width/height to match the OBS canvas (e.g. 1920 × 1080).
4. No custom CSS needed — the page is already full-bleed.

The page must work correctly when rendered in a headless Chromium context (which is what
OBS uses for Browser Sources). This means:
- No Web Audio, getUserMedia, or other APIs that require user interaction to activate.
- `window.innerWidth` and `window.innerHeight` reflect the Browser Source dimensions,
  not a physical monitor — the layout algorithm must rely on these, not on screen size.

---

## Lifecycle Diagram

```
Operator sets up OBS Browser Source (URL = /board/:collectionId)
      |
      v
Page loads → fetches available photos → shuffles → renders grid
      |
      v
Poll starts (every 5 s)  <--------------------------------------+
      |                                                         |
      v                                                         |
Poll fires → GET /collections/:collectionId/board              |
      |                                                         |
      +-- Photo set unchanged? → do nothing -------------------+
      |
      +-- Photo removed (card sold)?
            |
            v
      Remove photo, reshuffle, re-render grid ------------------+
      |
No more cards? No → continue polling --------------------------(above)
      |
      v
All cards sold → grid is empty → page shows nothing (or a placeholder)
```

---

## What Is Out of Scope for This Step

- The Spots Page UI for marking a spot as sold (that already exists or is planned
  separately).
- Authentication / access control for the board URL (the URL is treated as a secret
  share link for now).
- Animations or transitions between reshuffles (the grid simply re-renders; smooth
  animations can be added later).
- Multiple simultaneous Collections on one board (one Collection per board URL).
- Historical view of which cards have been sold.
