# Voyage — Offline-First Trip Companion

A standalone Progressive Web App (PWA) for keeping your whole trip in one place — **schedule, reservations, travel, and packing** — that works **completely offline** (built for a Disney cruise with no usable internet at sea).

The app ships with **zero personal data**. On first run it asks you to load a **trip file** that lives on *your* device. Everything is parsed and stored on-device in your browser. Nothing is ever uploaded — no accounts, no servers, no analytics, no CDNs.

> Voyage reuses the offline PWA architecture from the DCL Exchange Helper: a single self-contained `index.html`, a precaching service worker, and locally **vendored** pdf.js (no network at runtime).

---

## What's in this repo

```
index.html              The entire app (HTML + CSS + JS, no build step)
manifest.webmanifest    PWA manifest (installable, standalone)
sw.js                   Service worker — precaches the shell for offline use
pdf.min.mjs             Vendored pdf.js (PDF import) — no CDN
pdf.worker.min.mjs      Vendored pdf.js worker
icon-192.png            App icons
icon-512.png
apple-touch-icon.png
sample-trip.json        FAKE sample data (safe to commit / share)
README.md               This file
```

**Your real itinerary is never committed here.** Keep your real `trip.json` on your phone (Files app, iCloud, email-to-self, etc.) and load it into the app.

---

## Quick start (use it)

1. Open the app (your GitHub Pages URL, see below).
2. Tap **Upload trip file** and pick a `.json` (or a `.pdf` with embedded data), **or** tap *load a sample trip* to explore.
3. Use the bottom tabs: **Now · Schedule · Reservations · Travel · Packing**. The gear icon (top right) is Settings.
4. Edit anything — add/edit/delete events, reservations, travel legs, and packing items. Changes save instantly to this device.
5. In **Settings → Export JSON** or **Share with family** to hand the file to someone else, or **Print / Save readable PDF** for a printout you can also re-import later.

---

## Install to your iPhone home screen

1. Open the GitHub Pages URL in **Safari**.
2. Tap the **Share** button → **Add to Home Screen** → **Add**.
3. Launch it from the new icon — it opens full-screen (no browser bar).
4. Load your trip file once. From then on it works in **airplane mode**.

(Android/Chrome: menu → **Install app** / **Add to Home screen**.)

---

## Fork & host it on GitHub Pages

1. **Fork** this repo (or push these files to your own).
2. In your repo: **Settings → Pages**.
3. Under **Build and deployment**, set **Source: Deploy from a branch**, **Branch: `main`** (or your branch), **Folder: `/ (root)`**. Save.
4. Wait ~1 minute. Your app is live at:
   `https://<your-username>.github.io/<repo-name>/`

The app uses **relative paths** and a relative service-worker scope, so it works correctly from that `/<repo-name>/` subpath with no changes.

> After you change app code, bump the cache version in `sw.js` (`const CACHE = 'voyage-v8'` …) so installed copies pick up the update.

### Versioning

Voyage uses **date-based versions** (`YYYY.MM.DD`), in two places:

- **App version** — `APP_VERSION` in `index.html` is the date the app code last changed. Shown at the bottom of Settings. Update it (and bump the `sw.js` cache name) whenever you change app code.
- **Trip data version** — every export (JSON, share, or printable PDF) stamps a `meta` block into the file: `version` (`YYYY.MM.DD` of the save), `savedAt` (exact timestamp), and `app` (which app version wrote it). Settings also shows **when the last JSON was added** to this device. When family members trade files, the `meta.version` tells you which copy is newest.

---

## The trip data file

### Primary format: `trip.json`

This is the source of truth — plain JSON, parsed deterministically and offline.

```json
{
  "meta": {
    "version": "2026.07.02",
    "savedAt": "2026-07-02T12:00:00.000Z",
    "app": "2026.07.02"
  },
  "trip": {
    "name": "Sample Caribbean Cruise",
    "startDate": "2030-03-10",
    "endDate": "2030-03-15",
    "timezone": "America/New_York",
    "travelers": ["Traveler 1", "Traveler 2", "Traveler 3", "Traveler 4"]
  },
  "schedule": [
    {
      "id": "evt-001",
      "date": "2030-03-10",
      "time": "13:00",
      "endTime": "15:30",
      "title": "Embarkation",
      "type": "travel",
      "location": "Cruise Terminal",
      "confirmation": "",
      "notes": "Arrive by 12:30, passports + boarding pass ready",
      "done": false
    }
  ],
  "reservations": [
    {
      "id": "res-001",
      "category": "dining",
      "provider": "Disney",
      "title": "Palo Brunch",
      "date": "2030-03-12",
      "time": "11:30",
      "location": "Deck 12 Aft",
      "confirmation": "ABC123",
      "party": ["Traveler 1", "Traveler 2"],
      "cost": "$50/pp",
      "notes": "Dress code: no shorts"
    }
  ],
  "travel": [
    {
      "id": "trv-001",
      "type": "flight",
      "title": "Outbound flight",
      "date": "2030-03-09",
      "time": "07:00",
      "confirmation": "XXXXXX",
      "details": "Boarding 7:00 AM"
    }
  ],
  "packing": {
    "Traveler 1": [
      { "category": "Clothing", "items": [ { "id": "p1", "text": "Formal night outfit", "packed": false } ] }
    ],
    "Shared": []
  },
  "notes": "Freeform trip notes"
}
```

**Field reference**

| Section | Fields |
|---|---|
| `meta` | *(written automatically on export)* `version` (`YYYY.MM.DD` — the date this file was last saved), `savedAt` (full ISO timestamp), `app` (Voyage version that wrote it). Optional on hand-written files; lets you tell at a glance which of two shared files is newer. |
| `trip` | `name`, `startDate` (`YYYY-MM-DD`), `endDate`, `timezone` (IANA, e.g. `America/New_York`), `travelers` (array) |
| `schedule[]` | `id`, `date`, `time` (`HH:MM`), `endTime`, `title`, `type`, `location`, `confirmation`, `notes`, `done` |
| `reservations[]` | `id`, `category`, `provider`, `title`, `date`, `time`, `location`, `confirmation`, `party[]`, `cost`, `notes` |
| `travel[]` | `id`, `type`, `title`, `date`, `time`, `confirmation`, `details` |
| `packing` | object keyed by person → `[{ category, items: [{ id, text, packed }] }]` |
| `suggestions[]` | *(optional)* activity ideas you can browse per day and add to the schedule: `id`, `date`, `time`, `title`, `type`, `location`, `notes`, `cost` |
| `notes` | freeform string |

**Activity ideas.** Any `suggestions` whose `date` matches the day you're viewing on the **Schedule** tab show up behind a 💡 banner. Tap it to browse them and **Add** the ones you want — each becomes a normal, editable schedule event. Ideas stay in the list so each traveler can pick their own. Great for a port day or sea day where you're choosing among options.

**Add to your phone's calendar.** Every schedule event, reservation, and travel leg has a 📅 button that exports it as a standard `.ics` file — open it to add it to Apple/Google Calendar. **Settings → Add trip to Calendar** exports the whole trip at once. Times are written as floating local time (the clock value shown), generated entirely on-device with no account or network.

**Packing — copy to everyone.** Build one person's list, then use **Copy list to everyone** (bottom of their packing list) to push the common items (shorts, t-shirts, etc.) to every traveler. Existing items are kept; only missing ones are added, and each person tracks their own checkboxes.

**Allowed values**

- schedule `type`: `travel · dining · show · excursion · port · activity · reservation · other` (color-coded)
- reservation `category`: `dining · excursion · spa · tour · other`
- reservation `provider`: `Disney · independent`
- travel `type`: `flight · drive · hotel · transfer · other`

**Rules**

- Times are 24-hour `HH:MM`. Dates are `YYYY-MM-DD`.
- `id`s should be unique; if any record omits one, the app assigns it on import.
- The app **validates on import** and shows a friendly list of problems rather than crashing.
- Editing in the app and re-exporting **round-trips** losslessly (including `done`/`packed` state) — so progress travels with the file.

### Bridge format: a readable PDF with embedded data

If you'd rather carry a PDF you can also *read*, put a delimited data block anywhere in the PDF's text layer:

```
...your human-readable itinerary text...

===VOYAGE-DATA-START===
{ ...the same JSON as above... }
===VOYAGE-DATA-END===
```

On import the app extracts the PDF text, finds the block between the markers, and parses it. If the markers aren't present it tells you so. (It only parses the delimited block — it does **not** guess structure from arbitrary PDF prose.)

The app's own **Print / Save readable PDF** (Settings) produces exactly this kind of file: a clean printout **plus** an embedded data block, so a Voyage PDF you save is *both* readable *and* re-importable. (App-generated PDFs encode the block as base64 so PDF line-wrapping can't corrupt it; hand-written raw-JSON blocks are also accepted.)

---

## Sharing with family

- **Settings → Share with family** uses your device share sheet (AirDrop, Messages, email) to send the `trip.json`; if that's unavailable it copies the JSON to your clipboard.
- The recipient opens Voyage and taps **Upload trip file**.
- Because the file carries the full state, everyone can keep their own copy and re-share an updated one anytime.

---

## Offline check (airplane-mode test)

1. Open the app online once and **load a trip file** (so the shell *and* your data are cached locally).
2. Turn on **Airplane Mode** (or DevTools → Network → *Offline*).
3. Fully close and reopen the app.
4. Everything should still work: all tabs render, edits save, export/print work. The only things that need a network are the very first page load and pushing repo updates.

---

## Privacy & design principles

- **Private by architecture** — the repo holds only generic code + fake sample data. Your real data is runtime-loaded and never transmitted.
- **No runtime network calls** — no analytics, telemetry, CDNs, or APIs. pdf.js is vendored locally.
- **Local-only persistence** — your trip lives in this browser's `localStorage` and survives close/reopen.
- **Round-trippable** — export any time; your edits are never trapped.
- **Touch-friendly & sun-readable** — big tap targets, plus a high-contrast toggle in Settings.

---

## Suggested improvements baked in (beyond the original plan)

- **Unified timeline** — reservations and travel automatically appear on the day's schedule and feed the live **Now / Next** banner, so one glance covers everything, not just hand-entered schedule events.
- **One file that's readable *and* importable** — the generated PDF embeds a tamper-resistant (base64) data block, collapsing the two file formats into one.
- **Progress travels with the file** — `done` and `packed` flags export with the JSON, so a file you share already reflects where you are.
- **Ship-time aware** — Now/Next and the clock use the trip's IANA `timezone`, which matters when the ship's clock differs from your phone.
- **High-contrast mode** — for reading on a sunny pool deck.
