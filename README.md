# Exam Gate Scanner

An offline web app that checks student ID cards at the exam gate. It scans the QR code on the card (exactly 8 digits, the roll number), looks the roll up in the eligibility lists loaded on the phone, and shows a full-screen result with the student's time slot and exam. After setup it needs no internet.

| Screen | Meaning |
|---|---|
| Green: REGISTERED | Student is eligible. Shows `time:` and `exam:` (one block per exam). |
| Orange: DUPLICATE | This roll already entered on this phone. Shows the time of the first entry. |
| Red: NOT ELIGIBLE | Roll is in a list but marked as not eligible (for example Fail or Rejected). |
| Red: WRONG EXAM | Phone is set to one exam, but the student is eligible only for another. Shows that exam and time. |
| Red: NOT REGISTERED | Roll is not in any loaded list. Send to the help desk. |
| Blue: WRONG GATE | Roll is outside this phone's roll range (only if a range is set). |
| Grey: INVALID CODE | The QR is not exactly 8 digits. |
| Grey: LOAD LIST FIRST | No list is loaded on this phone. |

The red and blue screens offer **Admit anyway (approved)** for a coordinator-approved entry, and **Dismiss**.

## Files (keep all seven together)

`index.html`, `jsQR.js`, `sw.js`, `manifest.webmanifest`, `logo.png`, `icon-192.png`, `icon-512.png`

## 1. Prepare the lists (CSV)

The app reads your eligibility sheets directly, so you can use the Google Sheets exports as they are. Download each sheet with File > Download > Comma Separated Values (.csv).

**Exam name.** The app labels each file "Oral Viva" if the file name or the title rows contain "oral" or "viva", and "Written Exam" if they contain "written". Otherwise it uses the file name.

**How a file is read**
- The header row is the first row with a cell starting with `roll` (any capitals), for example `Roll No`. Title rows above it are fine.
- Next to each roll column, the app looks for a `Time Slots` column (header contains "time") and an `Eligibility` column (header contains "eligib"). The student name comes from a `Name` column (header contains "name"): the nearest one to the **left** of each Roll column, never past the previous Roll column. This matches sheets laid out as `S No | Students Name | Roll No | Time Slots | ...`.
- Several roll blocks side by side are all read. For example, an eligible table on the left and a Fail or Rejected table on the right.
- A row is eligible if its Eligibility text is `Eligible`, `Pass`, `Passed`, `Yes`, `OK`, `Registered` or `Selected`. Any other text, or an empty cell, counts as not eligible. If a block has no Eligibility column, every row in it is eligible.
- Roll numbers longer than 8 digits have their leading zeros stripped. Shorter ones are padded to 8 digits (`36857` becomes `00036857`).
- The same roll repeated in one exam is merged (an eligible row wins). The roll, name, time slot, exam and eligibility are kept. Scores are dropped when the file loads.

A plain CSV with one `roll` column also works. It gets no time slot, and the exam label is the file name.

**Never upload the CSV files to GitHub or Netlify.** They contain student names.

## 2. Put the app online

The app is hosted on Netlify, deployed from a GitHub repo. The repo can be private.

1. Create a GitHub repo (for example `gate-scanner`) and upload the seven files above at the top level.
2. In Netlify, choose Add new project > Import an existing project, connect GitHub, and pick the repo.
3. Leave **Build command**, **Publish directory** and **Functions directory** empty. The app is plain static files.
4. Click Deploy. Open the site link. You should see "Tap to start scanning".
5. **Hide the Netlify badge:** the free plan may show a "Powered by Netlify" badge that covers the Check button. In Netlify, go to Project configuration > General > Powered by Netlify badge and turn it off.
6. If you make the repo private and Netlify stops deploying, open GitHub Settings > Applications > Netlify > Configure and allow access to the repo.

The site link is public, but the app holds no student data. It cannot admit anyone without lists loaded on the phone.

## 3. Install on each phone (needs internet, once)

**Android (Chrome)**
1. Open the site link, tap "Tap to start scanning", and allow the camera.
2. Chrome menu > Install app (or Add to Home screen).
3. Open the app from the home-screen icon.

**iPhone (Safari)**
1. Open the site link in Safari, then Share > Add to Home Screen. The icon is the crest (the app sets an apple-touch-icon for this).
2. Open the app from the home-screen icon and allow the camera.
3. Do all further steps inside this icon app, not in Safari, because the two keep separate storage.

If the camera is blocked, allow it in the phone's browser settings for the site, then reload.

## 4. Load and check the lists

1. Send the CSV files to the phone (AirDrop, Drive, WhatsApp or cable). Delete them from chats afterwards.
2. In the app: Settings > **Load CSV files**, and select all the files at once. This replaces the current list. Use **Add more files** to merge extra files into the current list.
3. Read the summary under the buttons: total rolls, and for each file the eligible and not eligible counts. Compare it with your sheets.
4. Tap **View list** to see every roll with its time slot and exam. "NOT ELIGIBLE" marks the others. Use the search box to check specific rolls.
5. Set **Exam checked at this phone** to one exam (**pick one exam per phone**). "All exams" mode shows every exam a student is eligible for, and counts one entry per roll across exams.
6. Optional: **Gate name** (shown in the top bar) and **Roll range** (with 2 phones, give each its own range so one student can't be admitted at both).

## 5. Test before exam day

Make a small test CSV with about 16 rolls, including `00302025`, for the first round. Never put it in the repo.

| Test | Expected result |
|---|---|
| Scan a registered card | Green, with the right time and exam |
| Scan the same card again after moving it out of view for 2 seconds | Orange DUPLICATE with the first entry time |
| Reload the page, then scan the same card | Still orange |
| Settings > **Reset entries**, then scan it | Green again |
| Keep the card in view after a green result | Nothing new happens |
| Open Settings (or View list) with a card in view | Nothing is admitted. Close it: the card is not admitted behind it |
| Hold a card outside the white guide frame | Ignored |
| Card with a scratch, a slightly blurred printout, a laminated card under ceiling light | Reads within about 3 seconds, at least 19 of 20 tries |
| Scan a QR with other content (for example `https://x.com/12345678`) or a 6-digit code | Grey INVALID CODE |
| Type a roll in the manual box, for example `302027` | Padded to `00302027` and checked |
| Type a roll marked Fail or Rejected | Red NOT ELIGIBLE |
| Restart the camera or the app | Flash is OFF again |
| Phone set to one exam, roll eligible only for the other | Red WRONG EXAM, shows the other time and exam |
| Student eligible for both exams, phone on "All exams" | Green with two blocks |
| Tap **Admit anyway** on a red screen, then enter it again | DUPLICATE |
| Set a roll range, enter a roll outside it | Blue WRONG GATE |
| Lock the phone or switch apps, then return | "Camera stopped. Tap to restart." Restart works, no double scanning |
| Manual entry while the camera is stopped | Still works |
| Settings > **Wipe all data** | List, counts line and entries are cleared. Scanning shows LOAD LIST FIRST |
| Airplane mode, close the app, reopen it from the icon | Opens, keeps its lists, scans |

Real-world checks: scan 20 real cards (including worn or glossy ones). At least 19 should work on the first try, in about 3 seconds each. Test on the oldest phone you will use. Tap **Reset entries** when finished, so test scans don't cause DUPLICATE on exam day.

## 6. On exam day

1. Arrive 30 minutes early. Open the app, check the loaded count and the exam setting, and run one test scan. Then tap **Reset entries**.
2. Turn off auto-lock, set the screen brightness high, and bring a power bank. At night, use the **Flash** button (Android only, if the phone supports it). Flash always starts OFF after a restart, and on laminated or plastic-coated cards it can make glare worse, so turn it on only when the light is poor. If a card is hard to read, try the **Zoom** button (cycles 1x, 1.5x, 2x, remembered).
3. Hold the card inside the white guide frame, 15 to 25 cm from the camera. Avoid glare. Only the area inside the frame is read, and scanning pauses while Settings or the list viewer is open.
4. After a scan, move the card out of view before the next one. The same card is ignored until it leaves the frame for about a second.
5. For a red result, send the student to the help desk. If the coordinator approves entry, tap **Admit anyway (approved)**.
6. If a card won't scan, type the roll number in the manual box. It accepts 1 to 8 digits. QR codes must be exactly 8 digits, and anything else shows grey INVALID CODE.
7. If the camera stops (a call, an app switch), a "Camera stopped. Tap to restart." screen appears. Tap the button. Manual entry keeps working meanwhile.
8. Keep a printed copy of the lists as a backup.
9. After the exam: Settings > **Wipe all data**, and delete the CSV files from the phone.

## Queue

The queue numbers students as they arrive in the waiting area, lists who is waiting, and lets you mark each student as sent to the exam room. There is **one running queue per exam** (a Viva queue and a Written queue). The time slot does not affect the numbering, so every student in an exam has their own number.

**Setup:** use one phone for the queue, and set **Exam checked at this phone** to **All exams**. The queue is per phone. Two phones would each number from 1.

**How it works**
1. Scan each student as they enter the waiting area. The green screen shows the **No.**, the name, the roll, the time slot and the exam. Numbers run 1, 2, 3 and so on within each exam, in the order students are scanned.
2. Tap **Queue (n)** in the top bar and choose the exam tab (Viva or Written). The list shows each student's number, name and roll (last 3 digits in bold, to tell similar names apart), with the student's time slot as a small label. It opens at the first waiting student.
3. Call students by number and name, check the roll on the card, and tap **Sent**. The row dims and the button becomes **Undo**. Rows never move or change number. Marking Sent is also how you clear a student who was scanned by mistake.
4. Search by roll number to find a student's place.
5. **Written tab:** each row shows its Batch (queue number divided by the batch size, rounded up). The batch size defaults to 20 and can be changed. **Mark batch sent** marks every waiting student in the lowest batch that still has anyone waiting.
6. **Export PDF** (on each tab) saves `viva_student_queue.pdf` or `short_exam_queue.pdf`: a formatted A4 table with a title and the time it was made, then Queue No., Roll No. and Name for every queued student of that exam, in queue order (the Written file also has a Batch column). The header row repeats on every page, rows are striped, long names are trimmed with "...", and pages are numbered. On an iPhone the share sheet opens so you can save it to Files. Names are plain Latin text in the PDF. If a queued student has no name, the app takes it from the loaded list when the PDF is made.
7. Scanning is paused while the Queue, Settings or the list viewer is open.

**Admit anyway on a red screen:** the student is admitted and added to the queue of their exam, with the next number. If the roll is not in any list, a prompt asks which queue to use (Viva or Written), with a "Don't queue" button.

**Reset entries** clears the queue and the Sent marks too. Loading a new CSV keeps the queue until you reset or wipe. If an older version was installed, tap **Reset entries** once so old numbers are cleared.

### Queue tests

| Test | Expected result |
|---|---|
| Load a CSV with `Students Name` left of `Roll No` | View list shows each name under its roll |
| Scan viva students from different time slots | Green with No. 1, 2, 3 and so on, in scan order, with no repeats |
| Scan a written student | Written has its own numbers, starting at 1 |
| Scan the same card again | Orange DUPLICATE with the same No. |
| Tap Sent, then scan that card again | Orange says it was already sent |
| Tap Sent on a row | Row dims, button becomes Undo, counters change, other numbers stay |
| Tap Undo | Row is waiting again |
| Close the app, restart the phone, reopen in airplane mode | Queue and Sent marks are intact |
| Open Queue with a card in front of the camera | Nothing is admitted, and closing does not admit it |
| Two students with the same name | Told apart by roll, last 3 digits bold |
| Written student, batch size 20 | Batch 1 for numbers 1 to 20, Batch 2 from 21 |
| Mark batch sent, after tapping Sent on one student of that batch | The rest of that batch is marked, nobody from the next batch |
| Admit anyway on a roll that is in a list (any slot, or none) | Queued in that exam with the next number |
| Admit anyway on a roll that is not in any list | Asks Viva queue, Written queue or Don't queue |
| Search the queue by roll digits | Matches in that exam's queue. Letters cannot be typed |
| Reset entries | Queue is empty |
| Wipe all data | List, names and queue are gone |
| Export PDF on the Viva tab | `viva_student_queue.pdf`: one list, every queued viva student, in queue order |
| Export PDF on the Written tab | `short_exam_queue.pdf` with a Batch column |
| Export a list longer than one page | Header row repeats, no rows cut off, page numbers shown |
| Export in airplane mode, then open the file | Works offline and opens in a PDF viewer |
| Open a tab with no queued students | Export PDF is disabled |

## How scanning works

- The app uses the phone's built-in barcode detector when it has one (Chrome on Android usually does). It copes better with blur and damage.
- Without it (for example Safari on iPhone), the app falls back to a multi-pass jsQR decoder that tries several image clean-ups in turn: raw, contrast stretch, adaptive threshold, sharpen, and upscale, plus an inverted-colours pass.
- The camera asks for 1920x1080 and continuous focus when the phone supports them. Only the area inside the white guide frame is read.
- If something has been in the frame for about 4 seconds without being read, a hint says to tilt the card and move it 15 to 25 cm away.

## Settings reference

| Item | What it does |
|---|---|
| Gate name | Label shown in the top bar |
| Roll range | Cards outside the range show WRONG GATE |
| Exam checked at this phone | Pick one exam per phone, or "All exams" |
| Load CSV files | Reads one or more files and replaces the list |
| Add more files | Reads more files and merges them into the list |
| View list | Opens the list in the app, with search |
| Queue (top bar) | Opens the queue: one list per exam, Sent / Undo, Mark batch sent (Written), Export PDF |
| Reset entries | Forgets who has entered and clears the queue and Sent marks. Keeps the list |
| Wipe all data | Removes the list, names, entries, queue and file summary from this phone |

## Privacy

The phone stores the roll list (roll, name, exam, time slot, eligible or not), a small record of who has entered (roll, exam, time of first entry) so it can show DUPLICATE, and the queue (roll, name, exam, slot, queue number and Sent time). There is no scan history and no log. Nothing is sent anywhere, and the repo holds no student data. **Wipe all data** removes all of it.

The one way data leaves the phone is **Export PDF**, which you start yourself. The PDF contains names and roll numbers and is saved in the phone's Downloads (or Files). Delete these files after the exam.

## Updating the app

1. Edit the file and upload it to GitHub. Netlify redeploys automatically.
2. In `sw.js`, raise the version number (for example `gate-v12` to `gate-v13`).
3. Open the app on each phone with internet, twice. The first open downloads the update, the second runs it.
4. If you changed the logo or icons, remove the app from the home screen and add it again.

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera doesn't start (NotAllowedError) | Allow camera permission for the site in the browser settings, then reload. The page must use `https://`. |
| Camera stopped | Tap "Tap to restart". This happens after a call, an app switch or locking the phone. |
| Everyone shows INVALID CODE | The QR is not exactly 8 digits. Check what the card contains. Type the roll manually meanwhile. |
| Hard-to-scan cards (scratched, blurry, laminated) | Tilt the card slightly to avoid glare, hold it 15 to 25 cm away, try the Zoom button, avoid direct light on the card, wipe the card clean, and use manual entry as the last resort. |
| Names show as a dash (screen, queue or PDF) | The CSV has no `Name` column, its header does not contain "name", or the list was loaded before names were supported. Fix the sheet if needed and reload the CSV files. Queued students then show their names. |
| Everyone shows NOT REGISTERED | The list is missing or wrong. Reload the CSV files and use View list to check. |
| WRONG EXAM for many students | "Exam checked at this phone" is set to the wrong exam. |
| Netlify badge covers the Check button | Turn it off in Netlify: Project configuration > General > Powered by Netlify badge. |
| App won't open offline | Open it once online from the home-screen icon, then retry. |
| List disappeared | The browser cleared its storage. Reload the CSV files. Opening the app the day before helps prevent this. |
| Old version still showing | Raise the version in `sw.js` and open the app online twice. |
| iPhone app has no list | You loaded it in Safari. Load it again inside the home-screen icon app. |
| Netlify stopped deploying after making the repo private | Re-allow the repo under GitHub Settings > Applications > Netlify. |

## Limits

- Duplicate detection works per phone only. Use roll ranges, or one gate per phone.
- A QR scan proves the card, not the person. Check the physical card and the student.
- Late changes to the lists need fresh CSV files loaded on each phone.
- The queue lives on one phone only and cannot be merged. A scan by mistake stays in the queue until you mark it Sent.
- The app keeps no audit trail of overrides. If you need one, keep a paper log at the help desk.
