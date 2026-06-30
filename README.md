# AutoGrow

**[🌿 Open AutoGrow](https://137-labs.github.io/autogrow/)**

A single-file, offline-first grow tracker for home cultivators — cannabis, tomatoes, and basil. Plant it, log it, feed it, harvest it, cure it. No account, no server, no subscription. Everything runs from one HTML file in your browser, and your data never leaves your device.

## Why this exists

Most grow-tracking apps want an account, a cloud sync, a subscription tier, and a community feed. AutoGrow is the opposite: a single HTML file you can open in any browser, install to your home screen like a real app, and use completely offline forever. It's built for people who want serious cultivation tooling (VPD calculations, nutrient scheduling, multi-tent layouts) without handing their grow data to a server somewhere.

## Features

### 🗺️ Map
A visual, spatial layout of your tent — not a list. Choose from 1, 4, 6, 7 (U-shape), or 9-plant layouts that match real tent configurations. Pot sizes render at different visual sizes based on real footprint math, so the map actually reflects your physical space. Supports multiple tents, each with its own independent layout, light setup, and zone configuration.

### 🌱 Plants
Full plant records from seed to harvest: name, strain, plant type, start date, target/actual harvest date, and tent position. Supports three plant types with different growth logic:
- **Cannabis** — hard lifecycle (seed → veg → flower → harvest), escalating feed schedule, flush before harvest
- **Tomato** — grows once, then fruits indefinitely with a steady low-N/high-P-K feed
- **Basil** — grows once, then harvested as needed indefinitely with minimal feed

Each plant tracks its own log history: height, watering, feeding (with exact nutrient amounts), pH used, notes, and photos.

### 📋 Today
Your daily action list. Per-plant: current stage, day count, week number, watering schedule, light distance recommendations, and nutrient dosing — pulled from a real Canna Terra feed chart or your own custom nutrient bottles. Also includes:
- **VPD Calculator** — live vapor pressure deficit reading from your tent's temp/humidity, with a target range for the plant's current stage and a leaf-to-air temperature offset adjustment for different light intensities
- **Hero summary card** — days to next harvest, anything overdue in the drying rack, total cured weight on hand, all at a glance

### 🗄️ Pantry
Four sub-sections covering your entire post-veg pipeline:
- **Nutrients** — track what bottles you actually own, with fill percentage and low-stock warnings; doses are calculated only from products you actually have
- **Seed Vault** — your seed inventory with a built-in Royal Queen Seeds strain database (19 strains with realistic flowering times), or your own custom strains. Tracks germination success rate per strain over time, learned from your own grows
- **Drying Rack** — active drying batches with a day counter and an ideal 10–14 day window indicator. Supports manual entries (for plants that were already drying before you started using the app), clearly tagged as manual and excluded from grow-success stats
- **Cured Inventory** — what's jarred and on hand, with total weight tracked. Also supports manual entries with the same separation from real grow data

### ♻️ Perpetual
A staggered harvest planner for continuous-cycle growing. Configure how many perpetual slots you're running and the gap between plantings — only the first empty slot tells you to plant now, with the rest offset automatically so you're never planting everything at once.

### 📚 Learn
Built-in growing guides and reference material for each stage of cultivation.

### Multi-tent support
Run more than one tent at once. Each tent has its own light model/wattage/schedule, pot size, zone layout, VPD settings, and electricity cost tracking — fully independent, switchable from the UI.

### AI Grow Advisor
An optional advisor that analyzes your actual setup and plant data and gives beginner-friendly, specific advice — current plant health, what to do this week, concerns, and a tip. Requires an internet connection (it calls the Claude API directly); there is no offline AI mode, since a real local LLM isn't feasible inside a single HTML file.

### Reports & exports
- **Print-friendly grow report** — clean, formatted black-and-white printable summary per plant
- **CSV export** — full log history (every plant, every entry) for use in spreadsheets

### Electricity cost tracking
Enter your light's wattage and your local electricity rate, and the app calculates daily/weekly/monthly running cost automatically based on your light schedule.

### Themes
Garden Green (default) or OLED Black — true black backgrounds for battery savings and easier eyes in a dark tent at night.

### Installable PWA
Add it to your phone or desktop home screen for a standalone, full-screen app experience with no browser address bar. On Chrome/Edge/Android you'll get a native install prompt automatically the first time you open it with no data yet; on iOS Safari (which doesn't support that prompt) you'll get on-screen "Add to Home Screen" instructions instead.

## What it can't do

Being honest about the tradeoffs of a single-file, no-server app:

- **No cloud sync.** There's no account and no server. Your data lives only on the device you're using. See [Backup & Data Safety](#backup--data-safety) below — this is the one thing you actually have to manage yourself.
- **No multi-device sync.** You can move data between devices manually via the backup file, but there's no automatic syncing between a phone and a desktop.
- **No social/community features.** No public grow logs, no forums, no sharing feed. It's a private tool, by design.
- **No push notifications.** Installing to your home screen gives you a standalone app window, but not real background reminders — a true service worker (needed for background notifications) requires a same-origin `.js` file served over http(s); browsers reject that inside a single self-contained HTML file by spec.
- **AI Advisor requires internet.** It calls an external API live; there's no offline AI fallback.
- **Single user per device.** No login, no per-user accounts — whoever has the device has the data.

## Tech overview

- Pure HTML/CSS/JavaScript, zero build step, zero dependencies — one file
- Structured data (plants, logs, tents, pantry, etc.) lives in `localStorage`
- Photos live in `IndexedDB` (kept separate from localStorage since photos are too large for it at any real volume)
- An ES6 `Proxy` wraps tent-specific settings so older single-tent code paths keep working transparently after the multi-tent upgrade
- No backend, no database, no tracking, no analytics

## Backup & Data Safety

Because there's no server, **your device is the only copy of your data.** Everything below exists because of that one fact.

### Manual backup
Header 💾 icon, or Setup ⚙️ → **Download Full Backup**. Produces `autogrow-backup.json` containing everything — plants, logs, tents, pantry, seed vault, drying rack, cured inventory, and every photo (pulled from IndexedDB and embedded as base64). Past 150 photos, you'll be offered a faster data-only backup option to avoid freezing the tab on mobile.

### Auto-backup
- Fires automatically **at most once per real day** — but only on a day you actually take an action (add a plant, log water/feed, harvest, move to drying rack, jar & cure, delete, etc.).
- Just opening the app to look around does **not** trigger it. No data change → no save → no backup.
- Filename is always `autogrow-backup.json` (not date-stamped), so it doesn't pile up dozens of differently-named files in your downloads. The exact export timestamp is still recorded *inside* the file itself.

### First-backup nudge
If you have real grow data but have **never once** taken a backup, a popup asks you to back up right now. If dismissed, it reappears every 3 days until an actual backup happens — intentional, not a bug, since losing data with zero cloud safety net is the worst experience this app can deliver.

### Restore
Header 📂 icon, or Setup ⚙️ → **Restore from Backup File** → pick the `.json` file. Works on any device/browser — phone or desktop — as long as you have the backup file. This **completely replaces** all current data on the device; you'll be shown the backup's plant/photo count and export date before confirming, since it can't be undone.

### Reset App
Setup ⚙️ → **Danger Zone → Reset App**. Permanently wipes everything on the device — plants, logs, tents, pantry, seed vault, drying/cured inventory, every photo. Requires typing `DELETE` to confirm (not just OK/Cancel, since that's too easy to tap through by accident on mobile). No undo — back up first if you want to keep anything.

### Crash recovery
If something causes a display error, the app shows a "Something went wrong" screen instead of going blank — your saved data is untouched either way, but you get a one-tap "Download Backup Now" button right there before doing any troubleshooting.

### What this doesn't do
The app can't push backups to Google Drive/iCloud/etc. automatically — that requires an account and a server, which this app deliberately doesn't have. If your device is set to auto-sync its downloads folder to a cloud service, the backup file ends up off-device with zero extra effort; otherwise, it's on you to move the file somewhere safe occasionally.
