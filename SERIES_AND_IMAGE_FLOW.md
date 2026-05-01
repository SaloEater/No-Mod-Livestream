# Card Digitalization — Series & Image Flow

## Overview

When the operator starts scanning cards, the System creates a **Series** — a named, ordered record of every card scanned in that batch. The Series is the authoritative container for the scan. All captured images travel through the backend, which is responsible for storing them in the correct location; the desktop app is never the permanent home for any photo.

---

## What Is a Series?

A **Series** represents one scanning batch. It groups every approved card into an ordered list, preserving the sequence in which cards were photographed.

Key properties of a Series:

- **Unordered** — cards within a Series are not assigned a position; the set of photos matters, not their sequence.
- **Has a status** — a Series is either **Open** (scanning is still in progress) or **Closed** (the operator has confirmed the batch is complete and submitted it).
- **Is the source of truth** — once an image is accepted into a Series, the backend owns it. The app holds it temporarily only until the backend confirms receipt.

---

## Business Rules

### Creating a Series
- A new Series is created when the operator names it and clicks **"Create Series"** in the app.
- Only one Series can be **Open** at a time. If the operator reopens the app with an existing open Series, they resume it rather than starting a new one.

### Linking a Series to a Break
- After a Series is closed, the operator goes to the Spots Page and selects that Series as the source for a specific Break.
- This link (Break → Series) is what allows the Cards Board Page to resolve which photos to display for a given channel and Livestream.

### Adding Cards to a Series
- Each approved photo is saved locally and staged for upload.
- Retaken photos (where the operator chose **Retake**) are discarded and never enter the Series.

### Closing a Series
- The operator reviews the thumbnail grid and clicks **Send** when all cards have been scanned.
- Each staged photo is sent to the backend individually and registered as an entry in the Series.
- If the backend does not confirm receipt of a photo, the operator is shown an error and can retry the upload.
- Once the backend confirms all photos, the Series is closed and no further cards can be added.
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
Photo staged locally                |
      |                             |
More cards? -------------------------+
      |
      v
Operator presses Send
      |
      v
Photos sent to backend one by one
      |
      v
Backend confirms receipt
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
- Preventing a second open Series from being created while one is already open.
- Exposing the Series and its photos to the rest of the System once the Series is closed.

---

## What the App Is Responsible For

- Creating a new Series (or resuming an existing open one) when scanning starts.
- Staging each approved photo locally until the operator presses **Send**.
- Sending all staged photos to the backend when the operator presses **Send**.
- Displaying confirmation (or error) feedback after the upload attempt.
- Triggering the Series close once the backend confirms all photos.

---

## Photo Entity

Each photo belongs to a Series and has an optional name for internal accounting (not
displayed on the Cards Board). Photos also carry a sold status, which defaults to false
and is set to true by the operator during a livestream.

All photos in a closed Series that have not been marked sold are displayed on the Cards
Board Page. The operator marks a photo as sold by clicking it in the board's operator
mode. There is no link between Photos and Spots — the operator uses visual recognition
to match a pulled card to its photo on the board.
