# Automatic Daily Scout — setup

What this unlocks: leads get found and added to one running Google Sheet
every day automatically, with nobody opening a browser. Fundamentally
different from the manual "Export to Google Sheet" button, which needs a
person to click through Google's account picker every single time — that
approach can't run unattended because the login token it gets only lives a
few minutes and nothing is stored afterward.

Unattended automation needs a **Google Service Account** instead of the
one-click sign-in flow. A service account is its own permanent identity
(not tied to any person's Gmail) that our backend can authenticate as,
indefinitely, with no expiring token and no login screen involved.

## Setup (~10 minutes, one time)

### 1. Create the service account

1. Go to **console.cloud.google.com**, select the same project used for
   the existing MLF Sheets export (the one with the OAuth client already
   set up — check `~/code/cabinet/.env` for `GOOGLE_OAUTH_CLIENT_ID` to
   confirm you're in the right project).
2. **APIs & Services → Credentials → + CREATE CREDENTIALS → Service account**
3. Name it `mlf-daily-scout`. Skip the optional role-grant and
   user-access steps (not needed for this).
4. Click into the newly created service account → **Keys** tab →
   **Add Key → Create new key → JSON**. This downloads a `.json` file —
   treat it like a password, it's a permanent credential.

### 2. Create the running spreadsheet and share it

1. Go to **sheets.google.com**, create a new blank spreadsheet, name it
   "Modern Line Furniture — Daily Scout Log".
2. Add a header row matching the export columns: Company, Website,
   Category, Territory, Why it fits, Contact, Status, Assigned to, Assign
   note, Verified, Notes, Source, Date found.
3. Click **Share**, and share it with the service account's email address
   — it looks like `mlf-daily-scout@your-project.iam.gserviceaccount.com`
   (find it in the downloaded JSON under `client_email`, or on the service
   account's page in Cloud Console). Give it **Editor** access.
4. Copy the spreadsheet ID from its URL:
   `https://docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`

### 3. Send me two things

1. The downloaded JSON key file's contents (the whole file — it has
   `client_email` and `private_key` fields I need)
2. The spreadsheet ID from step 2.4

I'll set them as env vars on the igor-hq worker (`MLF_SCOUT_SA_EMAIL`,
`MLF_SCOUT_SA_PRIVATE_KEY`, `MLF_SCOUT_SHEET_ID`) and wire up the daily
cron job. Once that's done, this runs by itself every day going forward —
no further action needed, ever, unless the sheet needs to move.

## What runs daily once this is live

Every day: search all 5 categories nationwide → skip anything already in
the lead list (same dedup as the manual Daily Scout button) → save new
leads to the app like normal → append just the new rows to the Daily
Scout Log sheet (never overwrites past days, just adds to the bottom).
Nothing needs manual review unless you want to check the sheet.
