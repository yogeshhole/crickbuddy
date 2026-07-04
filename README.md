# Cricbots Weekend Cricket — site

A single-page booking site for the weekend cricket group: fixture list, UPI payment with QR, WhatsApp confirmation, live slot counts, and a passcode-gated admin dashboard.

## Files
- `index.html` — the whole app (UI + logic)
- `manifest.json` — PWA manifest (install to home screen)
- `sw.js` — service worker for offline app-shell caching
- `icons/icon-192.png`, `icons/icon-512.png` — placeholder app icons (swap these for your own logo)

Bookings are stored in **Firebase Firestore** (free "Spark" plan — no credit card required), so this works properly once you host it standalone on GitHub Pages. You need to create your own free Firebase project and paste its config in — takes about 10 minutes.

## 1. Create a free Firebase project
1. Go to https://console.firebase.google.com and sign in with a Google account.
2. Click **Add project**, give it a name (e.g. `cricbots`), and finish the wizard (you can skip Google Analytics).
3. In the left sidebar: **Build → Firestore Database → Create database**. Choose a nearby region, and start in **production mode** (we'll set our own rules below).
4. In the left sidebar: **Build → Authentication → Get started → Sign-in method → Email/Password → Enable**.
5. Still in Authentication, go to the **Users** tab → **Add user** → enter the email and password *you* (the admin) will use to log into the dashboard. This is the only account that can change booking statuses.
6. In the left sidebar: **Project settings (gear icon) → General → Your apps → Web (`</>`)**. Register an app (any nickname, no need to set up Hosting). Copy the `firebaseConfig` object it shows you.

## 2. Paste your config into `index.html`
Find the `CONFIG` block near the top of the `<script>` tag and replace the placeholder:

```js
const FIREBASE_CONFIG = {
  apiKey: "REPLACE_ME",
  authDomain: "REPLACE_ME.firebaseapp.com",
  projectId: "REPLACE_ME",
  storageBucket: "REPLACE_ME.appspot.com",
  messagingSenderId: "REPLACE_ME",
  appId: "REPLACE_ME"
};
```

Also set, in the same block:
```js
const UPI_ID = '9850946044@ybl';
const MATCH_FEE = 150;                 // per player per match
const TOTAL_SLOTS = 22;                // slots per match
const WHATSAPP_GROUP_LINK = 'https://chat.whatsapp.com/REPLACE_WITH_YOUR_GROUP_LINK';
const WHATSAPP_CONFIRM_NUMBER = '';    // optional: a number to DM confirmations to
```

The season window (1 Oct – 30 May) is calculated automatically, so fixtures regenerate correctly every year without further edits.

## 3. Lock down Firestore with security rules
In the Firebase console: **Firestore Database → Rules**, and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /bookings/{bookingId} {
      allow read: if true;                 // anyone can see live slot counts
      allow create: if request.resource.data.keys().hasAll(['name','phone','txnId','amount','date','submittedAt','status'])
                    && request.resource.data.status == 'pending';
      allow update: if request.auth != null;  // only the signed-in admin can change status
      allow delete: if false;
    }
  }
}
```

Click **Publish**. This means anyone can submit a booking (as intended — that's the public booking form), but only your admin login can mark a booking confirmed/cancelled.

## 4. How it works once live
- Every visitor's browser holds one real-time connection to the `bookings` collection, so slot counts and the admin dashboard update instantly everywhere, on every device.
- The admin dashboard sign-in (email + password) uses the account you created in step 1.5 — there's no separate app passcode anymore.
- Firebase's free Spark plan gives 50,000 reads and 20,000 writes a day, which comfortably covers a weekend cricket group.

## 5. Hosting on GitHub Pages
1. Create a new GitHub repo and add all these files (keep the folder structure, e.g. `icons/icon-192.png`).
2. Repo Settings → Pages → Deploy from branch → select `main` and `/ (root)`.
3. Your site will be live at `https://<username>.github.io/<repo>/`.

## 6. Hosting on Netlify
1. Drag the whole folder into Netlify's "Deploy manually" upload area (or connect the GitHub repo).
2. No build step is needed — it's static HTML/CSS/JS.

## Note
`FIREBASE_CONFIG` values (apiKey, etc.) are meant to be public — they identify your project, they don't grant access on their own. Access is controlled entirely by the security rules in step 3, so make sure those are in place before you publish the real link.
