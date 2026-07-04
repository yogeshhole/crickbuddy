# Cricbots Weekend Cricket — site

A single-page booking site for the weekend cricket group: fixture list, UPI payment with QR, WhatsApp confirmation, live slot counts, and a passcode-gated admin dashboard.

## Files
- `index.html` — the whole app (UI + logic)
- `manifest.json` — PWA manifest (install to home screen)
- `sw.js` — service worker for offline app-shell caching
- `icons/icon-192.png`, `icons/icon-512.png` — placeholder app icons (swap these for your own logo)

## Before you deploy — edit these in `index.html`
Look for the `CONFIG` block near the top of the `<script>` tag:

```js
const UPI_ID = '9850946044@ybl';
const MATCH_FEE = 150;                 // per player per match
const TOTAL_SLOTS = 22;                // slots per match
const WHATSAPP_GROUP_LINK = 'https://chat.whatsapp.com/REPLACE_WITH_YOUR_GROUP_LINK';
const WHATSAPP_CONFIRM_NUMBER = '';    // optional: a number to DM confirmations to, e.g. '919850946044'
const ADMIN_PASSCODE = 'cricbots2027'; // change this
```

- Set `WHATSAPP_GROUP_LINK` to your real group invite link.
- Change `ADMIN_PASSCODE` — this is a simple client-side gate, not real authentication. Don't use it to protect anything sensitive; it just keeps casual visitors out of the dashboard.
- The season is calculated automatically as the current or next "1 Oct – 30 May" window, so fixtures regenerate correctly every year without further edits.

## How bookings work
- Bookings are stored using the Claude artifact storage API, shared across everyone who opens the page (`bookings:<date>` keys). That means:
  - Anyone who opens the site can, in principle, see live slot counts — that's intended, so players know how full a match is.
  - Only the admin dashboard shows names, phone numbers, and transaction IDs — but technically the storage is shared, not access-controlled, so treat the passcode as a light deterrent rather than security.
- Because it's "last write wins," two people submitting at the exact same second on a nearly-full fixture could both get in even if that pushes past `TOTAL_SLOTS` by one. For a casual weekend game this is fine; for anything higher-stakes, verify manually in the admin dashboard before match day.

## Hosting on GitHub Pages
1. Create a new GitHub repo and add all these files (keep the folder structure, e.g. `icons/icon-192.png`).
2. Repo Settings → Pages → Deploy from branch → select `main` and `/ (root)`.
3. Your site will be live at `https://<username>.github.io/<repo>/`.

## Hosting on Netlify
1. Drag the whole folder into Netlify's "Deploy manually" upload area (or connect the GitHub repo).
2. No build step is needed — it's static HTML/CSS/JS.

## Notes on the storage-backed features
This app relies on `window.storage` (Claude's artifact persistence API), which only works when the page is opened as a Claude artifact — **not** once it's deployed standalone on GitHub Pages/Netlify. If you deploy outside Claude, bookings, live slot counts, and the admin dashboard will need a real backend (e.g. Firebase, Supabase, or a small serverless function) in place of the `getBookings` / `saveBookings` functions — everything else (layout, countdown, UPI/QR, WhatsApp links) will keep working as static HTML.
