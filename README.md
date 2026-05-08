# RevSync V3 — Setup Guide
## Mobius Revenue Intelligence Tool

---

## 📁 Files
- `index.html`     — Login page (Google OAuth)
- `dashboard.html` — Full dashboard with sidebar & history
- `README.md`      — This guide

---

## 🚀 Step 1 — Deploy to Netlify

1. Go to **netlify.com** → log in
2. Go to your existing site → **Deploys tab**
3. Drag the `revsync-v3` folder into the deploy box
4. Wait 15 seconds → your site updates at `https://mobiusvariancetool.netlify.app`

---

## 🔑 Step 2 — Google Sheets History Setup

This enables saving each month's analysis and browsing history inside RevSync.

### 2a — Create the Google Sheet
1. Go to **sheets.google.com** → create a new blank spreadsheet
2. Name the file: `RevSync History`
3. Rename the first tab (bottom of screen) to: `RevSync_History`
4. In Row 1, add these exact headers in columns A through I:
   ```
   id | monthLabel | savedOn | totalActual | totalForecast | netVariance | alertCount | threshold | resultJson
   ```
5. Copy the Sheet ID from the URL:
   `docs.google.com/spreadsheets/d/THIS_PART_HERE/edit`

### 2b — Make the Sheet accessible
1. Click **Share** (top right)
2. Change to **Anyone with the link → Viewer**
3. Click **Done**

### 2c — Create a Google Apps Script (for saving data)
Because the API key only allows reading, we use a free Apps Script to handle writes.

1. In your Google Sheet, click **Extensions → Apps Script**
2. Delete everything in the editor and paste this code:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.openById('YOUR_SHEET_ID').getSheetByName('RevSync_History');
  const data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    data.id, data.monthLabel, data.savedOn,
    data.totalActual, data.totalForecast, data.netVariance,
    data.alertCount, data.threshold, data.resultJson
  ]);
  return ContentService.createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doDelete(e) {
  const sheet = SpreadsheetApp.openById('YOUR_SHEET_ID').getSheetByName('RevSync_History');
  const id = e.parameter.id;
  const data = sheet.getDataRange().getValues();
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === id) { sheet.deleteRow(i + 1); break; }
  }
  return ContentService.createTextOutput(JSON.stringify({ status: 'deleted' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Replace `YOUR_SHEET_ID` with your actual Sheet ID
4. Click **Deploy → New deployment**
5. Type: **Web app**
6. Execute as: **Me**
7. Who has access: **Anyone**
8. Click **Deploy** → copy the **Web App URL**

### 2d — Create a Sheets API Key
1. Go to **console.cloud.google.com**
2. **APIs & Services → Library** → search **Google Sheets API** → Enable it
3. **APIs & Services → Credentials → Create Credentials → API Key**
4. Copy the API key

### 2e — Update dashboard.html
Open `dashboard.html` in Notepad → find `SHEETS_CONFIG` near the top:

```javascript
const SHEETS_CONFIG = {
  SHEET_ID: 'YOUR_SHEET_ID',           // ← paste your Sheet ID
  API_KEY: 'YOUR_SHEETS_API_KEY',       // ← paste your API key
  APPS_SCRIPT_URL: 'YOUR_APPS_SCRIPT_URL', // ← paste the Web App URL
  SHEET_NAME: 'RevSync_History',
};
```

Save → redeploy to Netlify.

---

## 👥 Adding / Removing Users

Open `index.html` in Notepad → find `ALLOWED_EMAILS`:

```javascript
const ALLOWED_EMAILS = [
  'hemamalini.mobius@gmail.com',
  // 'john@mobius.com',    ← add new users by removing the //
  // 'jane@mobius.com',
];
```

- To **add** a user: add their email inside quotes followed by a comma
- To **remove** a user: delete their line or add `//` at the start

Save → redeploy to Netlify.

---

## 📊 How to Use Monthly

1. Open `https://mobiusvariancetool.netlify.app`
2. Sign in with your Google account
3. In the **Overview** page, upload that month's Billing Advice + Forecast files
4. Select the month and set your variance threshold
5. Click **Run Analysis** — results appear instantly and are saved to history
6. Navigate via the sidebar:
   - **Variance Table** — full comparison with dept filter tabs
   - **Alerts** — projects exceeding threshold
   - **Unforecasted** — billed with no forecast
   - **Email Summary** — copy for email
   - **Download** — Excel / CSV exports
   - **All History** — browse past months, click any to view

---

## 🔄 Editing a Previous Month

1. Go to **All History** in the sidebar
2. Find the month → click **View**
3. Upload corrected files → Run Analysis again
4. It saves a new entry — you can delete the old one from History

---

## 📞 Support
Contact your IT admin or the tool developer to:
- Add new authorised users
- Reset the Google Sheets history
- Update the variance threshold default
