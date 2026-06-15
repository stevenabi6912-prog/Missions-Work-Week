# Scriptures Completed — live shared tracker

A single static HTML page that shows a **live, shared count** of John & Romans
Scripture booklets completed during the mission week. Anyone can open it on a
phone (no login) and see the same number update in real time. An admin (password
`5898`) can add/correct boxes; every change syncs to every device in 1–2 seconds.

- **Total Scriptures** = completed boxes × booklets per box (default **300**)
- **Average / hour** = total ÷ working hours elapsed since the mission start,
  counting **only weekday 9 AM–8 PM Eastern Time**.
- One file, no build step. Hosted on GitHub Pages. Shared state lives in
  **Firebase Realtime Database** (free tier).

Everything you edit lives in the clearly marked `CONFIGURE EVERYTHING HERE`
block near the top of [`index.html`](index.html).

---

## Part 1 — Create the Firebase project (5 minutes)

1. Go to **https://console.firebase.google.com** and sign in with any Google account.
2. Click **Add project** (or **Create a project**).
   - Name it e.g. `scripture-tracker`. Click **Continue**.
   - You can **turn OFF Google Analytics** (not needed). Click **Create project**, then **Continue**.
3. In the left sidebar, open **Build → Realtime Database**.
   - Click **Create Database**.
   - **Location:** pick **United States** (closest region).
   - For "Security rules" choose **Start in test mode** for now — we replace the
     rules in Part 3. Click **Enable**.
   - You'll land on the database screen. Note the URL at the top — it looks like
     `https://scripture-tracker-default-rtdb.firebaseio.com/`. You'll need it.

---

## Part 2 — Get your config keys and paste them in

1. In the sidebar click the **gear ⚙ → Project settings**.
2. Scroll to **Your apps** and click the **web icon `</>`**.
   - App nickname: `tracker`. **Do not** check "Firebase Hosting." Click **Register app**.
3. Firebase shows a `firebaseConfig = { … }` object. Copy those values.
4. Open [`index.html`](index.html), find **section 1) FIREBASE CONFIG**, and paste
   each value in. **Important:** make sure `databaseURL` is filled in with the URL
   from Part 1 (the auto-shown config sometimes omits it):

   ```js
   const FIREBASE_CONFIG = {
     apiKey:            "AIza...",
     authDomain:        "scripture-tracker.firebaseapp.com",
     databaseURL:       "https://scripture-tracker-default-rtdb.firebaseio.com",
     projectId:         "scripture-tracker",
     storageBucket:     "scripture-tracker.appspot.com",
     messagingSenderId: "1234567890",
     appId:             "1:1234567890:web:abc123"
   };
   ```

5. While you're in there, confirm the other config:
   - `ADMIN_PASSWORD` is `5898`
   - `DEFAULT_BOOKLETS_PER_BOX` is `300`
   - **`MISSION_START`** — set this to the **Monday your mission week starts**
     (year, month, day). It's currently `2026-06-15 9:00 AM ET`.

> **Note:** these Firebase web keys are *meant* to be public — they identify your
> project in the browser. What protects your data is the **security rules** in Part 3.

---

## Part 3 — Set the database security rules (public read, gated writes)

In the Firebase console: **Build → Realtime Database → Rules** tab. Replace
everything with this, then click **Publish**:

```json
{
  "rules": {
    "tracker": {
      ".read": true,
      ".write": true,
      "completedBoxes": { ".validate": "newData.isNumber() && newData.val() >= 0" },
      "bookletsPerBox": { ".validate": "newData.isNumber() && newData.val() >= 1" },
      "$other":         { ".validate": false }
    }
  }
}
```

What this does:
- **Anyone can read** the count (`".read": true`) — required so visitors see the
  number with no login.
- Writes are confined to the `tracker` node and **must be valid numbers** — random
  junk, strings, or extra fields are rejected, so the data can't be corrupted.

### About the admin password (read this)

Because GitHub Pages is static (no server you control), the `ADMIN_PASSWORD`
check happens **in the browser**. It hides the admin buttons from ordinary
visitors, which is all a trusted mission team needs. It is **not** cryptographic
protection — someone technical who reads the page source could still write to the
database. For a Scripture count that an admin can correct anytime, that's fine.

**Want genuinely locked-down writes?** Upgrade to Firebase Authentication — see
**Part 6 (optional)** at the bottom.

---

## Part 4 — Test it locally

You can't just double-click the file (Firebase needs an `http://` origin). From a
terminal in the `scripture-tracker` folder:

```bash
python3 -m http.server 8000
```

Open **http://localhost:8000** on your computer. You should see:
- the green **"Live — synced to all devices"** dot,
- the total showing `0`.

Tap **🔒 Admin**, enter `5898`, press **+1 Box** — the total should jump by 300.
Open the same URL on your phone at the same time to confirm it updates live on both.

---

## Part 5 — Deploy to GitHub Pages

The tracker is self-contained in the `scripture-tracker/` folder. The cleanest
way to host it is its **own repository** so its URL is simple and it's separate
from the Kingdom Clash game in this repo.

**Recommended — a dedicated repo:**

1. On GitHub, create a new repo, e.g. `scripture-tracker` (Public).
2. Upload the **contents** of this folder (`index.html`, `README.md`) to the repo
   root — so `index.html` sits at the top level.
3. In the repo: **Settings → Pages**. Under "Build and deployment", set
   **Source: Deploy from a branch**, **Branch: `main` / `(root)`**, **Save**.
4. Wait ~1 minute. Your live URL appears at the top of the Pages settings:
   `https://YOUR-USERNAME.github.io/scripture-tracker/`
5. Share that link. Everyone who opens it sees the same live count. 🎉

**Alternative — keep it in this repo:** push this folder, enable Pages on this
repo, and the tracker will live at
`https://YOUR-USERNAME.github.io/Kingdom-Clash-main/scripture-tracker/`.

---

## Daily use

- **Viewers:** just open the link. Big number = total Scriptures done. No login.
- **Admin:** tap **🔒 Admin → enter `5898`**, then:
  - **+1 Box** — one tap per finished box.
  - **Add a specific number of boxes** — e.g. type `5`, tap **Add**.
  - **Set total completed boxes** — overwrite with an exact count.
  - **− Remove 1 Box** — fix an over-count.
  - **Booklets per box** — change from 300 if a box size differs.
- Every admin action writes to Firebase, so **all open phones update within a
  second or two** — no refresh.

---

## Part 6 (optional) — Truly secure admin writes via Firebase Auth

If you want writes that *cannot* be forged by reading the page source:

1. Firebase console → **Build → Authentication → Get started →** enable
   **Email/Password**.
2. **Authentication → Users → Add user.** Use any email (e.g.
   `admin@yourteam.org`) and set the password to your master password.
3. Change the **Rules** to require login for writes:
   ```json
   {
     "rules": {
       "tracker": {
         ".read": true,
         ".write": "auth != null"
       }
     }
   }
   ```
4. Tell me you want the Auth version and I'll swap the password box in
   `index.html` for a real `signInWithEmailAndPassword` login (viewers still need
   no login — only writing requires it).

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Dot is red / "Firebase not configured" | Re-check every value in `FIREBASE_CONFIG`, especially `databaseURL`. |
| "Read error — check database rules" | Make sure the rules from Part 3 are **Published** with `".read": true`. |
| Admin save says "Save failed" | Rules must allow writes (Part 3) and you must be online (green dot). |
| Average shows `0` or `—` | The clock only counts weekday 9 AM–8 PM ET after `MISSION_START`. Outside that window it won't advance. |
| Number didn't change for others | Confirm they're on the same deployed URL and have a green "Live" dot. |
