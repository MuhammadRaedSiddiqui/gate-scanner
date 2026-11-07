# Exam Gate Scanner

An offline web app that checks student ID cards at the exam gate. It scans the QR code (an 8-digit roll number), checks it against a list of registered students loaded on the phone, and shows a full-screen result. After setup it needs no internet.

| Screen | Meaning |
|---|---|
| Green: REGISTERED | Student is on the list. Let them in. |
| Orange: DUPLICATE | This roll already entered on this phone. Shows the time. |
| Red: NOT REGISTERED | Not on the list. Send to the help desk. |
| Blue: WRONG GATE | Roll is outside this phone's range (only if a range is set). |
| Grey | No list loaded, or the QR is not a roll number. |

No student data is stored in this project. The roll list is loaded onto each phone separately and stays there. Nothing is recorded or exported: the phone only remembers who has entered, so it can show DUPLICATE.

## Files (keep all seven together)

`index.html`, `jsQR.js`, `sw.js`, `manifest.webmanifest`, `logo.png`, `icon-192.png`, `icon-512.png`

## 1. Prepare the roll list (CSV)

The CSV must have one column with the header `roll`, one 8-digit roll number per row (for example `00302025`), no duplicates.

1. In the Google Sheet linked to your registration Form, make an `Export` tab with `roll` in A1 and the formula below in A2. Change `B` to your roll number column and match your tab name.

   ```
   =ARRAYFORMULA(UNIQUE(FILTER(RIGHT("00000000"&REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""),8), LEN(REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""))>=1, LEN(REGEXREPLACE('Form Responses 1'!B2:B&"","[^0-9]",""))<=8)))
   ```
2. Check that the row count matches the number of valid registrations.
3. Download the `Export` tab: File > Download > Comma Separated Values (.csv). Name it `registered_YYYYMMDD.csv`.
4. Re-export after the registration deadline.

**Never upload the CSV to GitHub or Netlify.** It contains student data.

## 2. Put the app online

The app is hosted on Netlify, deployed from a GitHub repo. The repo can be private.

1. Create a GitHub repo (for example `gate-scanner`) and upload the seven files above at the top level of the repo.
2. In Netlify, choose Add new project > Import an existing project, connect GitHub, and pick the repo.
3. Leave **Build command**, **Publish directory** and **Functions directory** empty. The app is plain static files.
4. Click Deploy. When it finishes, open the site link. You should see "Tap to start scanning".
5. **Hide the Netlify badge:** the free plan may show a "Powered by Netlify" badge that covers the Check button. In Netlify, go to Project configuration > General > Powered by Netlify badge and turn it off.
6. If you make the GitHub repo private and Netlify stops deploying, open GitHub Settings > Applications > Netlify > Configure and allow access to the repo.

The site link is public, but the app contains no student data. It cannot admit anyone without a CSV loaded on the phone.

(GitHub Pages also works. In the repo, go to Settings > Pages and deploy from branch `main`, folder `/ (root)`. This needs a public repo on the free plan.)

## 3. Install on each phone (needs internet, once)

**Android (Chrome)**
1. Open the site link, tap "Tap to start scanning", and allow the camera.
2. Chrome menu > Install app (or Add to Home screen).
3. Open the app from the home-screen icon.

**iPhone (Safari)**
1. Open the site link in Safari, then Share > Add to Home Screen.
2. Open the app from the home-screen icon and allow the camera.
3. Do all further steps inside this icon app, not in Safari, because the two keep separate storage.

If the camera is blocked, allow it in the phone's browser settings for the site, then reload.

## 4. Load and check the roll list

1. Send the CSV to the phone (AirDrop, Drive, WhatsApp or cable). Delete it from chats afterwards.
2. In the app: Settings > **Load roll list (CSV)**. A line shows how many rolls loaded.
3. Tap **View list** to see every roll number, sorted. Use the search box to check that specific students are on the list. Compare the total with your Sheet.
4. Optional, in Settings:
   - **Gate name:** shown in the top bar.
   - **Roll range:** with 2 phones, give each one its own range (for example `00300000` to `00309999`), so the same student can't be admitted at both. Leave both fields empty to accept all rolls.

## 5. Test before exam day

A ready-made test file, `mock_registered.csv`, can be used for the first round (16 rolls including `00302025`). Never put it on the repo.

| Test | Expected result |
|---|---|
| Scan a registered card | Green REGISTERED |
| Scan it again after moving it out of view for 2 seconds | Orange DUPLICATE |
| Keep the card in view after a green result | Nothing new happens |
| Type a registered roll in the manual box | Green |
| Type the same roll again | Orange DUPLICATE |
| Type a roll without leading zeros (for example `302027`) | Green (padded to 8 digits) |
| Type a roll that is not on the list | Red NOT REGISTERED |
| Tap **Admit anyway** on a red screen, then type it again | DUPLICATE |
| Set a roll range and type a roll outside it | Blue WRONG GATE |
| Settings > **Wipe all data**, then scan | Grey LOAD LIST FIRST |

Then test in real conditions:

1. Load the real CSV.
2. Turn on airplane mode, close the app completely, and reopen it from the icon. It must open, show the loaded count, and scan.
3. Scan 20 real cards, including worn or glossy ones. At least 19 should scan on the first try, in about 3 seconds each.
4. Test on the oldest phone you will use.
5. Tap **Reset entries** in Settings when finished, so test scans don't show DUPLICATE on exam day.

## 6. On exam day

1. Arrive 30 minutes early. Open the app, check the loaded count, and run one test scan. Then tap **Reset entries**.
2. Turn off auto-lock and set the screen brightness high. Bring a power bank.
3. Hold the card 15 to 25 cm from the camera. Avoid glare.
4. After scanning a card, move it out of view before the next scan. The same card is ignored until it leaves the frame for about 1 second.
5. For a red result, send the student to the help desk. If the coordinator approves entry, tap **Admit anyway (approved)**.
6. If a card won't scan, type the roll number in the manual box.
7. Keep a printed copy of the registered list as a backup.
8. After the exam: Settings > **Wipe all data**, and delete the CSV from the phone.

## Settings reference

| Item | What it does |
|---|---|
| Gate name | Label shown in the top bar |
| Roll range | Cards outside the range show WRONG GATE |
| Load roll list (CSV) | Replaces the list on this phone |
| View list | Opens the list in the app, with search |
| Reset entries | Forgets who has entered, keeps the list |
| Wipe all data | Removes the list and entries from this phone |

## Updating the app

1. Edit the file and upload it to GitHub. Netlify redeploys automatically.
2. In `sw.js`, raise the version number (for example `gate-v4` to `gate-v5`).
3. Open the app on each phone with internet, twice. The first open downloads the update, the second runs it.
4. If you changed the logo or icons, remove the app from the home screen and add it again.

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera doesn't start (NotAllowedError) | Allow camera permission for the site in the browser settings, then reload. The page must be opened over `https://`. |
| Everyone shows NOT REGISTERED | The list is missing or wrong. Reload the CSV and use View list to check it. |
| Netlify badge covers the Check button | Turn it off in Netlify: Project configuration > General > Powered by Netlify badge. |
| App won't open offline | Open it once online from the home-screen icon, then retry. |
| List disappeared | The browser cleared its storage. Reload the CSV. Opening the app the day before helps prevent this. |
| Old version still showing | Raise the version in `sw.js` and open the app online twice. |
| iPhone app has no list | You loaded it in Safari. Load it again inside the home-screen icon app. |
| Netlify stopped deploying after making the repo private | Re-allow the repo under GitHub Settings > Applications > Netlify. |

## Limits

- Duplicate detection works per phone only. Use roll ranges or one gate per phone.
- A QR scan proves the card, not the person. Check the physical card and the student.
- Late registrations need a fresh CSV loaded on each phone.
