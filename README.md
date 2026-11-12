# Timesheet prototype — GitHub Pages setup

This version of the app talks to the Google Sheets API directly from the browser, with Google sign-in. Host it on GitHub Pages so the OAuth client has a real `https://` origin.

- App URL (after Pages is enabled): https://jasoncodematic.github.io/timesheet/
- Repository: https://github.com/jasoncodematic/timesheet
- Authorised JavaScript origin: `https://jasoncodematic.github.io` (no path)
- Spreadsheet: [Timesheet Prototype](https://docs.google.com/spreadsheets/d/1Kqn2XAPOD-R3O1TzVFMo8wT9q8FidRf2mjVdQqGdzlE/edit)
- Spreadsheet ID: `1Kqn2XAPOD-R3O1TzVFMo8wT9q8FidRf2mjVdQqGdzlE`
- Tab: `Entries` (columns: `id`, `date`, `description`, `active`, `createdAt`, `updatedAt`)

The page itself is public. Your data is not: only a Google account that you allow, and that can open the spreadsheet, can read or write it. The OAuth Client ID in the page is not a secret.

`manifest.json` and the icon PNG files are optional. The app works from `index.html` alone (demo mode works even as a local file). The manifest is there so **Add to Home Screen** looks decent.

## 1. Google Cloud project and the Sheets API

1. Open [Google Cloud Console](https://console.cloud.google.com/) and create a project, or pick an existing one.
2. Go to **APIs & Services > Library**, search for **Google Sheets API**, and click **Enable**.

## 2. OAuth consent screen

1. Go to **APIs & Services > OAuth consent screen**.
2. Choose **External** and create the app. App name can be “Timesheet”. Add your email as the support / developer contact.
3. Scopes: add
   - `https://www.googleapis.com/auth/spreadsheets` (sensitive) — lets the signed-in account read and write spreadsheets it already has access to, including this existing sheet, by ID.
   - `https://www.googleapis.com/auth/userinfo.email` (non-sensitive) — used only to show who is signed in and to hint that account on the next visit.
4. Stay in **Testing** mode and add `jason.mcculloch@gmail.com` as a test user. Only test users can sign in while the app is in Testing.
5. Why not `drive.file`? That scope only covers files this app created, or files the user opened with the Google Picker. It would not reach the existing Timesheet Prototype spreadsheet unless you added a picker step or created a new sheet.

### Testing vs publishing

- **Testing** is the right mode for a personal app. Add any other people you want to use it as test users (and share the spreadsheet with them).
- Google’s “refresh tokens expire after 7 days in Testing” caveat does **not** apply here. This app uses the Google Identity Services token client, which only receives short-lived access tokens (about an hour) and silently requests a new one with `prompt: ''` when needed. You may still see the account picker or consent screen again from time to time, especially after a long gap or if you revoke access.
- **Publishing** the consent screen to Production, with the sensitive `spreadsheets` scope, requires Google’s OAuth verification (privacy policy, demo video, and a review). Until that is approved, keep the app in Testing and add test users. Do not publish just to avoid the testing-user list.

## 3. OAuth Client ID (Web application)

1. Go to **APIs & Services > Credentials > Create credentials > OAuth client ID**.
2. Type: **Web application**. Name it “Timesheet web”.
3. Under **Authorised JavaScript origins**, add exactly these two origins, with no path:
   - `https://jasoncodematic.github.io`
   - `http://localhost:8080`
4. You do not need Authorised redirect URIs for this token-client flow.
5. Create the client and copy the **Client ID** (it looks like `123456789-abc.apps.googleusercontent.com`).

`http://127.0.0.1:8080` is a different origin from `localhost`. Use `http://localhost:8080` when testing locally, or add `127.0.0.1` as a third origin if you prefer that address.

## 4. GitHub Pages

The app files in this repo are `index.html`, plus optional `manifest.json` and icons.

If Pages is not on yet:

1. In the repo, go to **Settings > Pages**.
2. Under **Build and deployment**, set Source to **Deploy from a branch**.
3. Pick the default branch and `/ (root)`, then **Save**.
4. After a minute, the site is at https://jasoncodematic.github.io/timesheet/

On your phone, open that URL and use **Add to Home Screen**. Every push to the default branch updates the site in a minute or so.

## 5. Connect the app

1. Open https://jasoncodematic.github.io/timesheet/ (or the local URL below). It starts in **Demo mode**.
2. Open Settings and paste the OAuth Client ID. The spreadsheet ID is already filled in.
3. Save, then **Sign in with Google** using a test-user account that can edit the spreadsheet.
4. The SAMPLE rows from the sheet should appear. Adds, edits, deletes and restores are shown immediately and then saved in the background.

You can also hardcode `DEFAULT_CLIENT_ID` (and, if you ever change sheet, `DEFAULT_SPREADSHEET_ID`) at the top of the script in `index.html` before you publish, so Settings is pre-filled on every device.

To go back to browser-only data, open Settings and choose **Use demo mode**. That keeps your Client ID and spreadsheet ID so you can switch back.

## 6. Test locally

From the folder that contains `index.html`:

```bash
python3 -m http.server 8080
```

Open http://localhost:8080/ — not a `file://` URL. Google sign-in only works on an authorised origin. Demo mode still works if you open the file directly.

## 7. Turn off the old Apps Script deployment

The v1 `/exec` Web App URL was deployed with “Who has access: Anyone”, so anyone who has that URL can read and write the sheet. Once this Pages app is working, undeploy it:

1. Open the spreadsheet, then **Extensions > Apps Script**.
2. **Deploy > Manage deployments**, and archive or delete the Web app deployment.

You can leave `Code.gs` in the script project; it is simply unused.

## Limits and caveats

- The signed-in Google account must have edit access to the spreadsheet.
- Access tokens live in memory only (about one hour). The only thing stored on the device from sign-in is an email hint, used to pick the same account next time. Sign out clears that hint and the cached entry list.
- If a silent token refresh cannot open a popup, click **Sign in** again.
- After you edit `index.html`, commit and push; wait for Pages to update.
- Google Sheets API quotas apply (typically hundreds of requests per minute per user). This app is well within that for personal use.
- Deletes are still soft-deletes: the row stays in the sheet with `active` set to `FALSE`. Restore sets it back to `TRUE`.
- Date-only values are written as `YYYY-MM-DD` and displayed in `en-NZ`. The spreadsheet timezone should stay on Pacific/Auckland.
