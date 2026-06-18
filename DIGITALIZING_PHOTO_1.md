# Card Digitalization — Workflow

## Overview

Before cards are packed into boxes, each card must be photographed and linked to a Series in the System. A Series is a named group of card photos for a single mystery box set. The Python desktop app handles the full process: defining the series, scanning cards, and uploading photos asynchronously — so the human can start packaging immediately after pressing Send.

---

## Hardware Setup

- **Camera:** Phone or webcam on an overhead fixed mount, pointing straight down
- **Capture surface:** A card-sized lit area (lightbox or white mat) on the desk below the camera
- **Trigger:** A physical button (USB button, keyboard key, or foot pedal)
- **Display:** A monitor showing live preview and capture feedback

**Estimated cost:** ~$20–80

---

## Human Workflow

### Step 1 — Create a Series
1. Open the Python app
2. Type a series name (e.g., "March Prizm Lot") and click **"Create Series"**
3. App registers the series with the System and enters scanning mode

### Step 2 — Scan Cards
1. Pick up a card from the current series
2. Place it face-up on the capture surface
3. Check the monitor — preview shows **"Ready"** or **"Adjust"**, with the detected card bounds overlaid so the operator can verify the crop area before capturing
4. Press the button to capture
5. App automatically crops the captured image to the card bounds before displaying it
6. App shows the processed photos and allow to choose from **Approve** or **Retake**:
   - **Approve:** Photo is staged; move on
   - **Retake:** Photo discarded; reposition and capture again
   - **Set name:** Optionally rename the card before approving
   - **Set price:** Optionally set the price for the card before approving
7. User can select a team's name assigned for the card  
9. Place the card into the box and repeat for every card

### Step 3 — Send & Package
11. After all cards are scanned, app shows a thumbnail grid with a count
12. Optionally rename the series, then click **"Send"**
13. Upload starts in the background — minimize the app and begin sealing and shuffling boxes
14. App shows **"Upload complete"** notification when done

---

## Human Timeline

```
[Create series] -> [Scan all cards] -> [Press Send] -> [Package & shuffle boxes]
                                               |
                                   [Upload runs in background]
```

---

## Local Storage & Crash Safety

The app must store scanned images on the local disk immediately after capture so that a crash, accidental close, or power loss never forces the operator to rescan cards.

### How it works

- **Auto-save on approve** — as soon as the operator presses **Approve**, the cropped image is written to a local folder on the computer.
- **Local record** — the app maintains a record of the series name and the list of approved photos alongside the images.
- **Resume on launch** — when the app starts, it detects any incomplete series (approved images present but not yet uploaded) and offers the operator the option to **Resume** the previous series, skipping straight to the thumbnail grid with all previously scanned cards already loaded.
- **No duplicates** — photos that were already uploaded successfully are tracked so they are not re-uploaded if the series is resumed after a partial upload.

### Failure scenarios covered

| Event | Result |
|---|---|
| App closed before pressing **Send** | Reopen → Resume prompt → all scanned cards present |
| Crash during scan loop | Reopen → Resume prompt → only cards scanned before crash present |
| Crash mid-upload | Reopen → Resume prompt → already-uploaded images skipped, rest retried |
| Normal completion | Local folder cleaned up after a period of time (see Open Decisions) |

---

## System Requirements

The System needs full CRUD support for two new entities:

**Series** — create, list, view, rename, delete

**Photos** — upload to a series, list, delete, mark as sold

Photos become available on the Cards Board Page once the Series is closed. All photos in the Series are displayed on the board until the operator marks them sold during the livestream.
