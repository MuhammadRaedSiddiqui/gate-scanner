# Exam Gate Scanner

An offline web app that checks student ID cards at the exam gate. It scans the QR code (an 8-digit roll number), checks it against a list of registered students loaded on the phone, and shows a full-screen result. After setup it needs no internet.

| Screen | Meaning |
|---|---|
| Green: REGISTERED | Student is on the list. Let them in. |
| Orange: DUPLICATE | This roll already entered on this phone. Shows the time. |
| Red: NOT REGISTERED | Not on the list. Send to the help desk. |
| Blue: WRONG GATE | Roll is outside this phone's range (only if a range is set). |
| Grey | No list loaded, or the QR is not a roll number. |

No student data is stored in this project. The roll list is loaded onto each phone separately and stays there.

## Files (keep all six together)

`index.html`, `jsQR.js`, `sw.js`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`

## 1. Prepare the roll list (CSV)

The CSV must have one column with the header `roll`, one 8-digit roll number per row (for example `00302025`), no duplicates.

1. In the Google Sheet linked to your registration Form, make an `Export` tab with the formula below in A2 and `roll` in A1. Change `B` to your roll number column and match your tab name.

   ```
   =ARRAYFORMULA(UNIQUE(FILTER(RIGHT("00000000"&REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""),8), LEN(REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""))>=1, LEN(REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""))<=8)))
   ```
2. Check that the row count matches the number of valid registrations.
3. Download the `Export` tab: File > Download > Comma Separated Values (.csv). Name it `registered_YYYYMMDD.csv`.
4. Re-export after the registration deadline.

**Do not upload the CSV to GitHub.** It contains student data.

## 2. Put the app online (GitHub Pages)

1. On GitHub, create a new **Public** repository (for example `gate-scanner`).
2. Choose Add file > Upload files, add the six files above, and commit.
3. Go to Settings > Pages. Set Source to **Deploy from a branch**, branch `main`, folder `/ (root)`, and save.
4. After 1 to 2 minutes, open `https://YOUR-USERNAME.github.io/gate-scanner/`. You should see "Tap to start scanning".

## 3. Install on each phone (needs internet, once)

**Android (Chrome)**
1. Open the link, tap "Tap to start scanning", and allow the camera.
2. Chrome menu > Install app (or Add to Home screen).
3. Open the app from the home-screen icon.

**iPhone (Safari)**
1. Open the link in Safari, then Share > Add to Home Screen.
2. Open the app from the home-screen icon and allow the camera.
3. Do all further steps inside this icon app, not in Safari, because the two keep separate storage.

## 4. Load the roll list on each phone

1. Send the CSV to the phone (AirDrop, Drive, WhatsApp or cable). Delete it from chats afterwards.
2. In the app: Settings > **Load roll list (CSV)**.
3. Check that the count shown matches your Sheet.
4. Optional, in Settings:
   - **Gate name:** shown in the top bar.
   - **Roll range:** with 2 phones, give each one its own range (for example `00300000` to `00309999`), so the same student can't be admitted at both. Leave both fields empty to accept all rolls.

## 5. Test before exam day

1. Turn on airplane mode, close the app completely, and reopen it from the icon. It must open and show the loaded count.
2. Scan one registered card (green), one unregistered card (red), and the same registered card again after moving it out of view (orange).
3. Try the manual entry box and "Undo last scan".
4. **Settings > Reset entries** when finished, so test scans don't show DUPLICATE on exam day.

Pass condition: at least 19 of 20 real cards scan on the first try, in under 3 seconds each, in airplane mode.

## 6. On exam day

1. Arrive 30 minutes early. Open the app, check the loaded count, and run test scans.
2. Turn off auto-lock and set the screen brightness high. Bring a power bank.
3. Hold the card 15 to 25 cm from the camera. Avoid glare.
4. After scanning a card, move it out of view before the next scan. The same card is ignored until it leaves the frame for about 1 second.
5. For a red result, send the student to the help desk. If the coordinator approves entry, tap **Admit anyway (approved)**.
6. If a card won't scan, type the roll number in the manual box.
7. Keep a printed copy of the registered list as a backup.
8. After the exam: Settings > **Wipe all data**, and delete the CSV from the phone.

## Updating the app

1. Edit the file and re-upload it to GitHub.
2. In `sw.js`, raise the version number (`gate-v2` to `gate-v3`).
3. Open the app on each phone with internet, twice. The first open downloads the update, the second runs it.

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera doesn't start | Allow camera permission in the phone's settings. The page must be opened over `https://`. |
| Everyone shows NOT REGISTERED | The list is missing or wrong. Reload the CSV and check the count. |
| Page shows 404 on GitHub | Wait 2 minutes, check that Pages is enabled, and that `index.html` is in the repo root. |
| App won't open offline | Open it once online from the home-screen icon, then retry. |
| List or entries disappeared | The browser cleared its storage. Reload the CSV. Opening the app the day before helps prevent this. |
| Old version still showing | Raise the version in `sw.js` and open the app online twice. |
| iPhone app has no list | You loaded it in Safari. Load it again inside the home-screen icon app. |

## Limits

- Duplicate detection works per phone only. Use roll ranges or one gate per phone.
- A QR scan proves the card, not the person. Check the physical card and the student.
- Late registrations need a fresh CSV loaded on each phone.
