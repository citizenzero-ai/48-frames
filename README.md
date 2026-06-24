# 48 Frames — site + baked-in CRM

Three files, no build step, no server:

| File | What it is |
|---|---|
| `index.html` | Public landing page. The free-shoot form captures leads and redirects to the thank-you page. |
| `thank-you.html` | Confirmation page. Greets the lead by brand name (passed in the URL). |
| `crm.html` | Your private lead board. Review, status-track, note, and export every lead. |

## How a booking flows

1. Visitor fills out the **Claim my free set** form on `index.html`.
2. The lead is saved to the browser (`localStorage` key `48f_leads`) — and, if you set an endpoint, also sent to your Google Sheet.
3. They're redirected to `thank-you.html?brand=…`.
4. You open `crm.html`, enter your passphrase, and work the leads.

## The one thing to understand: where leads live

By default, leads are saved to **the browser they were submitted in**. That's perfect for testing and for leads *you* enter, but a real prospect filling out the form on their own phone saves the lead to *their* browser — you won't see it in your `crm.html`.

Two ways to handle that:

- **Backup / Import** (built in): on any machine, use **Backup** in the CRM to download a JSON file, and **Import** on another machine to merge it in. Good for moving your own data between computers.
- **CRM_ENDPOINT** (recommended for real lead-gen): point the form at a free Google Sheet so every visitor's lead lands in one place you control. Setup below.

## Set up a real lead inbox (free Google Sheet) — ~5 minutes

1. Create a new Google Sheet. In row 1, add headers: `createdAt  name  email  brand  status  notes  utm_source  utm_medium  utm_campaign  referrer  source  id`
2. In the sheet: **Extensions → Apps Script**. Paste this and Save:

   ```javascript
   function doPost(e){
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
     const d = JSON.parse(e.postData.contents);
     sheet.appendRow([
       d.createdAt, d.name, d.email, d.brand, d.status, d.notes,
       d.utm_source||'', d.utm_medium||'', d.utm_campaign||'',
       d.referrer||'', d.source||'', d.id
     ]);
     return ContentService.createTextOutput('ok');
   }
   ```
3. **Deploy → New deployment → Web app.** Set **Execute as: Me**, **Who has access: Anyone**. Deploy and copy the **Web app URL**.
4. Open `index.html`, find `const CRM_ENDPOINT = "";` near the bottom, and paste your URL between the quotes.

Now every booking writes a row to your sheet *and* still saves locally as a backup. (The form sends as `text/plain` so the request works without CORS setup — the Apps Script reads `e.postData.contents`.)

> Prefer Formspree, Zapier, or another tool? Any endpoint that accepts a POST with a JSON body works — just paste its URL into `CRM_ENDPOINT`.

## Your CRM passphrase

`crm.html` opens behind a passphrase. Open the file, find `const PASSPHRASE = "frames48";` and change it.

**Be aware:** this is a client-side gate — a deterrent, not real security. Anyone who views the page source can read the phrase. It keeps the board from opening by accident; it does not protect sensitive data on a public URL. If you need real protection, host `crm.html` somewhere with actual auth, or keep it off the public site and open it locally.

## Deploy on GitHub Pages

1. Put all four files in your repo root.
2. **Settings → Pages →** deploy from your `main` branch, root.
3. Your site is at `https://<user>.github.io/<repo>/` — the board is at `…/crm.html`.

## Quick test

1. Open the site, submit the form → you should land on the thank-you page with your brand name in the greeting.
2. Open `crm.html`, enter the passphrase → your test lead is in the list.
3. Change a status, type a note (it autosaves), then **Export CSV**.
