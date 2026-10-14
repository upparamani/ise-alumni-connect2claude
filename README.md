# ISE Alumni Connect — setup guide

**How it fits together (all free):**

| Piece | What it does |
|---|---|
| **GitHub Pages** | Hosts the two web pages (`index.html`, `admin.html`) at a public link you can share. |
| **Google Apps Script** (`Code.gs`) | The backend. Checks usernames/passwords, applies privacy rules, reads/writes the Sheet. |
| **Google Sheet** (`Alumni_Master_Database.xlsx` → Google Sheets) | The database. Lives in your Google Drive, private to you. |

The pages never touch the Sheet directly. They only call the Apps Script web app, which decides what each person may see or change.

---

## 1. Create the database (Google Drive)
1. Go to <https://drive.google.com> → **New → File upload** → choose `Alumni_Master_Database.xlsx`.
2. Open it → **File → Save as Google Sheets**. Rename it e.g. *ISE Alumni Database*.
3. Keep it private (Share → only you / named faculty). Alumni never open it; they use the website.

## 2. Add the backend
1. In that Google Sheet: **Extensions → Apps Script**.
2. Delete the sample code, paste the whole of `Code.gs`, click **Save**.
3. Pick the function **`setup`** in the toolbar → **Run**. Approve the permissions
   (*Advanced → Go to project (unsafe) → Allow* — this warning is normal for your own script).
4. **Deploy → New deployment → ⚙ Select type: Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - **Deploy**, then copy the **Web app URL** (ends in `/exec`).

> Whenever you later change `Code.gs`: **Deploy → Manage deployments → ✏ Edit → Version: New version → Deploy**. The URL stays the same.

## 3. Connect the pages
Open `index.html` and `admin.html` in Notepad and find this line in each (search for `API_URL`):

```js
const API_URL = '';
```
Paste your URL between the quotes in **both** files, e.g. `const API_URL = 'https://script.google.com/macros/s/AKfy…/exec';` and save.

## 4. Publish on GitHub Pages
1. <https://github.com> → **New repository** (e.g. `ise-alumni`, Public).
2. **Add file → Upload files** → upload **only** `index.html` and `admin.html` → Commit.
   Do **not** upload `demo-seed.js`, `Code.gs` or the `.xlsx`.
3. **Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save.**
4. After a minute your site is at `https://<your-github-username>.github.io/ise-alumni/` — that is the link you share.
   Faculty dashboard: `…/ise-alumni/admin.html`.

## 5. First use
1. Open `admin.html` on your site → passcode **`gitise2026`** → immediately click **⚙ Change passcode**.
2. Click **🔑 Generate login invites for everyone not yet activated**. A window lists every alumnus with a username and one-time temporary password, plus ready-to-send messages (copy / WhatsApp / email / CSV).
   - The sheet has no phone numbers or emails yet. Add them with **✏️ Edit** to get one-click WhatsApp/Email buttons, or just use **Copy message**.
   - Passwords are stored as hashes only, so each temporary password is visible **only at that moment**. Send them privately and don't keep the list.
3. Alumni open your link → **🔑 Alumni Login** → enter the username/temporary password → choose a new password → update their profile and choose Public / Semi-Anonymous / Stealth.
4. New alumni can use **+ Register as Alumnus**. Their entry waits in the dashboard for **Approve** (to publish instantly instead, set `AUTO_APPROVE: true` in `Code.gs`).

## Trying it on your computer first (demo mode)
Keep `index.html`, `admin.html` and `demo-seed.js` in one folder and leave `API_URL` empty. Open `index.html`. You will see the 20 seeded alumni; logins, edits and approvals work, but the data lives only in that browser. Demo admin passcode: `gitise2026`.

## Security notes (what is and isn't protected)
- Passwords: per-user random salt + SHA-256, never stored or logged in plain text. Alumni may choose any username/password (no length rule), so a very short password is easy to guess; the 5-attempt lockout limits the damage. The admin passcode must be at least 8 characters. 5 wrong attempts lock that account (or the admin login) for 15 minutes. Logins last 2 hours.
- Privacy tiers are enforced **on the server**: Stealth profiles and every contact detail (phone, email, USN) are never sent to the public page; Semi-Anonymous profiles have name, company and LinkedIn removed before sending.
- The admin passcode is checked on the server, not stored in the web page. Change the default on first login.
- The Apps Script URL is public by design (the website must call it), so rely on the passcode/lockout, keep the Sheet private, and keep `Code.gs` out of GitHub (it contains the default passcode).
- It is a good fit for a department directory. It is not a bank-grade identity system (single-round salted SHA-256, no email verification or "forgot password" — admins reissue a login with 🔑).
- The **Audit** tab records logins, edits, approvals and resets (never passwords).
