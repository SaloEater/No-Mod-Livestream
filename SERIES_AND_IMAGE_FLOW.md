# Card Digitalization — Series & Image Flow

## Overview

When the operator starts scanning cards, the System creates a **Series** — a named, ordered record of every card scanned in that batch. The Series is the authoritative container for the scan. All captured images travel through the backend, which is responsible for storing them in the correct location; the desktop app is never the permanent home for any photo.

---

## What Is a Series?

A **Series** represents one scanning batch. It groups every approved card into an ordered list, preserving the sequence in which cards were photographed.

Key properties of a Series:

- **Ordered** — cards are numbered in the order they were approved. This order is meaningful and must be preserved.
- **Has a status** — a Series is either **Open** (scanning is still in progress) or **Closed** (the operator has confirmed the batch is complete and submitted it).
- **Is the source of truth** — once an image is accepted into a Series, the backend owns it. The app holds it temporarily only until the backend confirms receipt.

---

## Business Rules

### Creating a Series
- A new Series is created when the operator names it and clicks **"Create Series"** in the app.
- Only one Series can be **Open** at a time. If the operator reopens the app with an existing open Series, they resume it rather than starting a new one.

### Adding Cards to a Series
- Each approved photo is immediately forwarded to the backend and registered as a new entry in the open Series.
- The backend assigns each entry a position number within the Series.
- If the backend does not confirm receipt, the photo is not considered part of the Series — the operator is shown an error and can retake or retry.
- Retaken photos (where the operator chose **Retake**) are never sent to the backend and never enter the Series.

### Closing a Series
- The operator closes the Series by pressing **Send** in the app.
- Once closed, no further cards can be added to it.
- The backend finalises the Series, making all its photos available in the System.

---

## Lifecycle

```
Series created
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
Photos available in System
```

---

## What the Backend Is Responsible For

- Accepting incoming images from the app and storing them in the correct place.
- Confirming receipt to the app so the operator knows the card is safely recorded.
- Maintaining the order of cards within the Series.
- Preventing a second open Series from being created while one is already open.
- Exposing the Series and its photos to the rest of the System once the Series is closed.

---

## What the App Is Responsible For

- Creating a new Series (or resuming an existing open one) when scanning starts.
- Sending each approved photo to the backend immediately after approval.
- Displaying confirmation (or error) feedback to the operator per card.
- Triggering the Series close when the operator presses **Send**.
- Holding the image locally only as a temporary buffer until the backend confirms receipt.
