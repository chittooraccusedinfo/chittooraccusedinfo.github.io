# Chittoor Accused Information System (CAIS)

Website: **https://chittooraccusedinfo.github.io**
Backend: Google Apps Script + Google Sheet (`ALL CHITTOOR ACCUSED INFORMATION`)

```
CAIS/
├── apps-script/          → paste into your Google Apps Script project
│   ├── Code.gs           (server: login, single-device sessions, search, audit log, JSON API)
│   ├── index.html        (same page – lets the /exec link also work as a backup)
│   └── appsscript.json   (manifest – time zone, web-app access)
└── github-pages/         → upload to the chittooraccusedinfo.github.io repository
    ├── index.html        (the website)
    └── robots.txt        (keeps the site out of Google search results)
```

---

## Step 1 – Google Apps Script (backend)

1. Open the Google Sheet → **Extensions → Apps Script**.
2. Replace the content of `Code.gs` with `apps-script/Code.gs`.
3. Create/replace an HTML file named **`index`** with `apps-script/index.html`.
4. (Optional) **Project Settings → Show "appsscript.json"** → replace with `apps-script/appsscript.json`.
5. Check the config at the top of `Code.gs`:
   - `SHEET_ID` – your sheet ID (already filled)
   - `DATA_SHEET_NAME = 'CTR_ACCUSED'`, `LOGIN_SHEET_NAME = 'LOGIN'`
6. Select the function `testConnection` → **Run** → allow permissions. In the log, check that `columnMap.ps` is not `-1`.
   (The data sheet needs a column named *Police Station*, *PS* or *PS Name*.)
7. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone** ← required, otherwise GitHub Pages cannot reach the script
8. Copy the **Web app URL** (ends with `/exec`).

> After every change to `Code.gs`: **Deploy → Manage deployments → ✏ Edit → Version: New version → Deploy**.
> This keeps the same `/exec` URL, so the website does not need changing.

Test: open `YOUR_EXEC_URL?api=ping` → it should show `"CAIS API is running"`.

## Step 2 – GitHub Pages (website)

1. Open `github-pages/index.html` and find this line (near the start of the `<script>`):
   ```js
   const API_URL = 'PASTE_YOUR_APPS_SCRIPT_EXEC_URL_HERE';
   ```
   Paste your `/exec` URL between the quotes.
2. In the GitHub repository **chittooraccusedinfo/chittooraccusedinfo.github.io**, upload
   `index.html` and `robots.txt` to the root (Add file → Upload files → Commit).
3. **Settings → Pages** → Source: *Deploy from a branch* → `main` / `(root)`.
4. Open https://chittooraccusedinfo.github.io after 1–2 minutes (press Ctrl+F5 to refresh).

## Sheets created automatically

| Sheet | Purpose |
|---|---|
| `LOGIN` | Username, Password, Name – only these users can log in |
| `ACTIVE_SESSIONS` | Who is logged in right now, device, time. **Delete a row to force-logout that user.** |
| `LOGIN_HISTORY` | Audit log: login, blocked login, logout, search, export, expiry |

## Single-device rule

- A user ID can be active on **one device only**. A second device gets the *"User Already Logged In"* popup with device and time.
- **Auto logout after 10 minutes of no response.** If the user does not click, type or scroll for 10 min, a warning with a 60-second countdown appears at 9 min, then the user is logged out. If the device is closed, offline or the phone screen is locked, the server logs the user out after 10 min with no response (about 90 s if the tab was closed normally).
- Logging in again from the **same browser** replaces the old session.
- Maximum 8 h per login.
- Change these in `Code.gs`: `IDLE_TIMEOUT_MIN`, `USER_IDLE_LOGOUT_MIN`, `CLOSE_GRACE_SEC`, `MAX_SESSION_HOURS`; and in `index.html`: `USER_IDLE_LOGOUT_MIN` (keep both at the same value).

## Admin tools (run from Apps Script editor)

- `forceLogoutAll` – logs out every user.
- `cleanupSessions` – add as a time-driven trigger (every 5 min) to keep `ACTIVE_SESSIONS` tidy.
- `clearMetaCache` – refreshes the police-station list after adding new stations.

## Important

- The GitHub page itself is public; data is only returned after a valid login.
- Passwords are stored as plain text in the `LOGIN` sheet – share that sheet only with administrators.
- Login-page portraits are built into `index.html`. To use other photos, put them in a Drive folder and set `PHOTO_FOLDER_ID` in `Code.gs`.
