# Card Digitalization — Step 2: Series & Image Flow

## Overview

When the operator begins scanning cards for a Collection, the System opens a **Series** — a named, ordered record of every card scanned in that session. The Series is the authoritative container for the scan batch. All captured images travel through the backend, which is responsible for storing them in the correct location; the desktop app is never the permanent home for any photo.

---

## What Is a Series?

A **Series** represents one scanning session for a Collection. It groups every card scanned in a single sitting into an ordered list, preserving the sequence in which cards were photographed.

Key properties of a Series:

- **Belongs to one Collection** — a Collection can have multiple Series over time (e.g. if scanning is split across multiple days or operators), but each Series belongs to exactly one Collection.
- **Ordered** — cards within a Series are numbered in the order they were approved. This order is meaningful and must be preserved.
- **Has a status** — a Series is either **Open** (scanning is still in progress) or **Closed** (the operator has confirmed the batch is complete and submitted it).
- **Is the source of truth** — once an image is accepted into a Series, the backend owns it. The app holds it temporarily only until the backend confirms receipt.

---

## Business Rules

### Creating a Series
- A new Series is opened automatically when the operator starts scanning cards for a Collection.
- Only one Series per Collection can be **Open** at a time. If the operator returns to a Collection that already has an open Series, they resume it rather than starting a new one.

### Adding Cards to a Series
- Each approved photo is immediately forwarded to the backend and registered as a new entry in the open Series.
- The backend assigns each entry a position number within the Series.
- If the backend does not confirm receipt, the photo is not considered part of the Series — the operator is shown an error and can retake or retry.
- Retaken photos (where the operator chose **Retake**) are never sent to the backend and never enter the Series.

### Closing a Series
- The operator closes the Series by pressing **Send** in the app.
- Once closed, no further cards can be added to it.
- The backend processes and finalises the Series, making all its photos available to the Collection.

### Multiple Series in One Collection
- A Collection accumulates cards across all its Series. If a Collection has Series A (20 cards) and Series B (15 cards), the Collection contains 35 cards total.
- Series are visible in the Collection view so the operator can distinguish batches (e.g. "scanned Monday" vs "scanned Tuesday").

---

## Lifecycle

```
Collection created
        |
        v
  Series opened  <--------------------+
        |                             |
        v                             |
  Card scanned & approved             |
        |                             |
        v                             |
  Image sent to backend               |
        |                             |
  Backend confirms receipt            |
        |                             |
  Card registered in Series           |
        |                             |
  More cards? -------------------------+
        |
        v
  Operator presses Send
        |
        v
  Series closed
        |
        v
  Photos available in Collection
```

---

## What the Backend Is Responsible For

- Accepting incoming images from the app and storing them in the correct place within the Collection/Series structure.
- Confirming receipt to the app so the operator knows the card is safely recorded.
- Maintaining the order of cards within each Series.
- Preventing a second open Series from being created for a Collection that already has one.
- Exposing the Series and its photos to the rest of the System once the Series is closed.

---

## What the App Is Responsible For

- Opening a new Series (or resuming an existing open one) when scanning starts.
- Sending each approved photo to the backend immediately after approval.
- Displaying confirmation (or error) feedback to the operator per card.
- Triggering the Series close when the operator presses **Send**.
- Holding the image locally only as a temporary buffer until the backend confirms receipt.
