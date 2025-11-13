# AMSAG Logbook Database

A lightweight Flask web application for managing AMSAG vehicle logbooks. Import data from spreadsheets or Google Sheets, search by owner/logbook/rego, edit ownership, cancel logbooks, and export to CSV. The interface uses the AMSAG colour palette so it can be embedded into [amsag.com.au](https://www.amsag.com.au).

## Features

- 🔎 Search by owner name, logbook number, or registration.
- ➕ Create new logbooks directly from the dashboard.
- ✏️ Edit ownership details, notes, and vehicle information.
- 🚫 Mark logbooks as cancelled or reactivate them later.
- 📥 Import from CSV/Excel uploads or by pasting a Google Sheet link.
- 📤 Export the entire database as a CSV download.
- 📱 Responsive design that can be embedded in an iframe.

## Getting started

1. **Create a virtual environment (optional but recommended):**
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **(Optional) Configure hosted Google Sheet + URL prefix:**
   ```bash
   export GOOGLE_SHEET_URL="https://docs.google.com/.../export?format=csv"
   export LOGBOOK_APP_PREFIX="/logbookdatabase"
   ```

   - `GOOGLE_SHEET_URL` pre-populates the import page and enables the "Sync from AMSAG master sheet" shortcut.
   - `LOGBOOK_APP_PREFIX` ensures links and static assets work when the site is reverse-proxied beneath `/logbookdatabase`.

4. **Run the application:**
   ```bash
   flask --app app run
   ```

   The site will be available at <http://127.0.0.1:5000/>.

5. **Embed in the AMSAG website:**
   Use an `<iframe>` pointing to the deployed application. The layout is responsive and designed to match the AMSAG colour scheme.
   ```html
   <iframe
       src="https://www.amsag.com.au/logbookdatabase/"
       title="AMSAG Logbook Database"
       style="width: 100%; min-height: 900px; border: 0;"
       loading="lazy"
       allowfullscreen>
   </iframe>
   ```

## Data import format

Your CSV/Excel/Google Sheet should include headers with the following column names (case-insensitive; spaces and punctuation are ignored):

| Column          | Required | Description                                 |
|-----------------|----------|---------------------------------------------|
| `logbook_number`| ✅        | Unique identifier for the logbook           |
| `owner_name`    | ✅        | Current owner's full name                   |
| `rego`          | ✅        | Registration number                         |
| `vehicle`       | ❌        | Vehicle description (make/model/year etc.)  |
| `notes`         | ❌        | Additional freeform notes                   |
| `status`        | ❌        | Either `active` or `cancelled` (defaults to active)

Common variations such as **Logbook Number**, **Owner Name**, **Registration Number**, **Vehicle Details**, **Comments**, or **Logbook Status** are recognised automatically, so you can keep the Google Sheet headers human-friendly without breaking the import.

Existing logbooks are updated when their logbook number already exists in the database.

## Google Sheet importing tips

- Make sure the sheet is shared with "Anyone with the link" or your service account (if running privately).
- Paste the share link into the import form. The app automatically converts it to the correct CSV export URL.
- Only the first worksheet is imported for Excel files; ensure your data sits on the first sheet.

## Database

Logbooks are stored in `logbooks.db` (SQLite). The database is created automatically when the server starts.

To start with a clean slate, stop the server and delete the `logbooks.db` file.

## Deployment + embedding notes

- The Flask secret key defaults to `amsag-logbook-secret`. Override this with the `FLASK_SECRET_KEY` environment variable in production.
- File uploads are stored temporarily in the `uploads/` directory during import and deleted once processing completes.
- When reverse proxying behind `/logbookdatabase`, set `LOGBOOK_APP_PREFIX=/logbookdatabase` **or** add the `X-Script-Name: /logbookdatabase` header so generated links stay within the iframe.
- If the iframe lives on a different domain, ensure the hosting service allows embedding (no `X-Frame-Options: DENY`). Most static hosts let you disable that header via configuration.
