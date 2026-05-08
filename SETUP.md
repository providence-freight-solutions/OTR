# OTR Dispatch Portal — Setup Guide

Two files make up the entire system:

| File | Where it lives |
|------|---------------|
| `index.html` | Your GitHub repository (served via GitHub Pages) |
| `Code.gs` | Inside your Google Sheet (via Apps Script) |

---

## Step 1 — Add the Apps Script to your Google Sheet

1. Open your Google Sheet (the one with the **LOAD BOARD** tab)
2. Click **Extensions → Apps Script**
3. Delete any code already in the editor
4. Open `Code.gs` from this repo and paste the entire contents in
5. Click **Save** (the floppy disk icon), name the project anything you like (e.g. `OTR Dispatch`)

---

## Step 2 — Deploy the Apps Script as a Web App

1. In the Apps Script editor, click **Deploy → New deployment**
2. Click the gear icon next to **Select type** and choose **Web app**
3. Fill in the settings:
   - **Description:** OTR Dispatch Portal
   - **Execute as:** Me *(your Google account)*
   - **Who has access:** Anyone
4. Click **Deploy**
5. Google will ask you to authorize the script — click through the prompts and allow access
6. After deploying, you'll see a **Web app URL** — it looks like:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```
   **Copy this URL** — you'll need it in Step 3.

> ⚠️ **Important:** Every time you edit `Code.gs`, you must create a **New deployment** (not update the existing one) to get your changes to take effect. The old URL keeps working until you choose to archive it.

---

## Step 3 — Configure index.html

1. Open `index.html` in a text editor
2. Find this line near the top of the `<script>` section:
   ```js
   const APPS_SCRIPT_URL = "YOUR_APPS_SCRIPT_URL_HERE";
   ```
3. Replace `YOUR_APPS_SCRIPT_URL_HERE` with the URL you copied in Step 2:
   ```js
   const APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycb.../exec";
   ```
4. While you're there, you can also change the admin password:
   ```js
   const ADMIN_PASSWORD = "dispatch2026";
   ```
   Make sure this matches the `ADMIN_PASSWORD` in `Code.gs`.

---

## Step 4 — Push to GitHub Pages

1. Create a GitHub repository (public or private — both work with Pages)
2. Add `index.html` to the repo root
3. Go to **Settings → Pages**
4. Under **Source**, select **Deploy from a branch** → **main** → **/ (root)**
5. Click **Save**
6. After a minute or two, your site will be live at:
   ```
   https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/
   ```

---

## Step 5 — Add the Users tab to your Google Sheet

The app creates the **Users** tab automatically the first time a driver registers. It will appear at the far right of your tab list so it doesn't interfere with your existing tabs.

The Users tab has three columns:

| username | password_hash | loads |
|----------|---------------|-------|
| john.smith | a3f5c... | 5992,5994 |

- **username** — generated from their full name (e.g. `john.smith`)
- **password_hash** — SHA-256 hash of their password (never stored in plain text)
- **loads** — comma-separated load numbers you've assigned to them

You can inspect and edit this tab directly in Google Sheets if needed, but the app manages it automatically.

---

## How to use the portal

### As a dispatcher (Admin)
- Go to your GitHub Pages URL
- Log in with username `admin` and your dispatch password
- **All Loads tab** — see every load pulled live from your Google Sheet, click any to view the full manifest
- **Drivers & Assignments tab** — see registered drivers, click **Assign Loads** to give a driver access to specific loads, or **Remove** to delete their account
- Click **↻ Refresh** any time to pull the latest data from your Google Sheet

### As a driver
- Go to the same GitHub Pages URL
- Click **Create Driver Account**, enter name and password
- Tell your dispatcher your username so they can assign your loads
- Log in to see only your assigned loads
- Tap any load to see the full delivery manifest with stops, orders, weights, pallets, and pay

---

## Updating your load data

The app reads directly from your Google Sheet every time someone logs in or clicks Refresh. **You don't need to do anything in the app** — just update your sheet as normal and the drivers will see the latest data.

---

## Security notes

- Passwords are hashed with SHA-256 before being stored in Google Sheets — plain text passwords are never saved anywhere
- The Apps Script URL is not a secret (it only reads your sheet and manages the Users tab), but you should still avoid posting it publicly
- The admin password is stored in plain text in `index.html` and `Code.gs` — for a higher-security setup you could move it to Apps Script Properties, but for an internal tool this is generally fine
- Drivers can only see loads that have been explicitly assigned to them — the app never exposes other loads on the client side

---

## Troubleshooting

**"Could not connect" or loads not loading**
- Make sure your Apps Script is deployed with **Who has access: Anyone**
- Check that the URL in `index.html` matches the deployment URL exactly
- After editing `Code.gs`, always create a **New deployment** — don't just save the file

**"Tab 'LOAD BOARD' not found"**
- Make sure your sheet's first tab is named exactly `LOAD BOARD` (all caps, with a space)

**Loads appear but data looks wrong**
- The parser detects rows dynamically by looking for the CUSTOMER/SHIPPER header row and the LOAD NUMBER footer row — as long as your sheet structure stays consistent, it will adapt automatically to rows moving up and down

**Driver can't log in after registering**
- Check the Users tab in Google Sheets — their row should be there
- If the Users tab doesn't exist, the first registration creates it automatically
