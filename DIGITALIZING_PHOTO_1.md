# Card Digitalization — Workflow

## Overview

Before cards are packed into boxes, each card must be photographed and linked to a Collection in the System. A Collection is a named group of card photos for a single mystery box set. The Python desktop app handles the full process: defining the collection, scanning cards, and uploading photos asynchronously — so the human can start packaging immediately after pressing Send.

---

## Hardware Setup

- **Camera:** Phone or webcam on an overhead fixed mount, pointing straight down
- **Capture surface:** A card-sized lit area (lightbox or white mat) on the desk below the camera
- **Trigger:** A physical button (USB button, keyboard key, or foot pedal)
- **Display:** A monitor showing live preview and capture feedback

**Estimated cost:** ~$20–80

---

## Human Workflow

### Step 1 — Create a Collection
1. Open the Python app
2. Type a collection name (e.g., "March Prizm Lot") and click **"Create Collection"**
3. App registers the collection with the System and enters scanning mode

### Step 2 — Scan Cards
4. Pick up a card from the current collection
5. Place it face-up on the capture surface
6. Check the monitor — preview shows **"Ready"** or **"Adjust"**, with the detected card bounds overlaid so the operator can verify the crop area before capturing
7. Press the button to capture
8. App automatically crops the captured image to the card bounds before displaying it
9. App shows the processed photo — press **Approve** or **Retake**
   - **Approve:** Photo is staged; move on
   - **Retake:** Photo discarded; reposition and capture again
10. Place the card into the box and repeat for every card

### Step 3 — Send & Package
11. After all cards are scanned, app shows a thumbnail grid with a count
12. Optionally rename the collection, then click **"Send"**
13. Upload starts in the background — minimize the app and begin sealing and shuffling boxes
14. App shows **"Upload complete"** notification when done

---

## Human Timeline

```
[Create collection] -> [Scan all cards] -> [Press Send] -> [Package & shuffle boxes]
                                                   |
                                       [Upload runs in background]
```

---

## Local Storage & Crash Safety

The app must store scanned images on the local disk immediately after capture so that a crash, accidental close, or power loss never forces the operator to rescan cards.

### How it works

- **Auto-save on approve** — as soon as the operator presses **Approve**, the cropped image is written to a local session folder (e.g. `~/NoModScans/<collection-name>/<timestamp>.jpg`).
- **Session manifest** — alongside the images, the app maintains a lightweight JSON file (e.g. `session.json`) that records the collection name, creation time, and the list of approved image paths in order.
- **Resume on launch** — when the app starts, it detects any incomplete session (approved images present but not yet uploaded) and offers the operator the option to **Resume** the previous session, skipping straight to the thumbnail grid with all previously scanned cards already loaded.
- **No duplicates** — images that were already uploaded successfully are marked in the manifest so they are not re-uploaded if the session is resumed after a partial upload.

### Session folder structure

```
~/NoModScans/
  March-Prizm-Lot_2024-03-15T10-22/
    session.json        ← manifest (collection name, status, image list)
    0001_1710494520.jpg
    0002_1710494535.jpg
    ...
```

### Failure scenarios covered

| Event | Result |
|---|---|
| App closed before pressing **Send** | Reopen → Resume prompt → all scanned cards present |
| Crash during scan loop | Reopen → Resume prompt → only cards scanned before crash present |
| Crash mid-upload | Reopen → Resume prompt → already-uploaded images skipped, rest retried |
| Normal completion | Session folder kept for 7 days then auto-deleted |

---

## System Requirements

The System needs full CRUD support for two new entities:

**Collections** — create, list, view, rename, delete

**Photos** — upload to a collection, list, delete, mark as valuable

Photos become available on the Spots Page once uploaded, where the operator flags which ones contain a valuable card for display on the Cards Board Page.
