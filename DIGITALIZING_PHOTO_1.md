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
6. Check the monitor — preview shows **"Ready"** or **"Adjust"**
7. Press the button to capture
8. App shows the processed photo — press **Approve** or **Retake**
   - **Approve:** Photo is staged; move on
   - **Retake:** Photo discarded; reposition and capture again
9. Place the card into the box and repeat for every card

### Step 3 — Send & Package
10. After all cards are scanned, app shows a thumbnail grid with a count
11. Optionally rename the collection, then click **"Send"**
12. Upload starts in the background — minimize the app and begin sealing and shuffling boxes
13. App shows **"Upload complete"** notification when done

---

## Human Timeline

```
[Create collection] -> [Scan all cards] -> [Press Send] -> [Package & shuffle boxes]
                                                   |
                                       [Upload runs in background]
```

---

## System Requirements

The System needs full CRUD support for two new entities:

**Collections** — create, list, view, rename, delete

**Photos** — upload to a collection, list, delete, mark as valuable

Photos become available on the Spots Page once uploaded, where the operator flags which ones contain a valuable card for display on the Cards Board Page.
