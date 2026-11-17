# Video evidence (mp4 only)

Shared rules for recording and storing test evidence. The same file ships in `qase-run` and `test-task`; keep both copies identical.

## Environment
- Headless only. Sequential runs, at most 2 workers. Do not use the claude-in-chrome extension (low-RAM machine).
- Testing environment URL, VPN note and test accounts: from the Tracking sheet (`Config` tab and `Account Test` tab). Never print a password, never type it on screen in a recording.
- Log in **before** recording starts (reuse a saved Playwright `storageState` file under the scratchpad) so the login form never appears in a video.

## Record
- One video per test unit (one Qase case, or one AC/EC). Start recording at the first step, stop after the last Expected Result is visible.
- Playwright `recordVideo` writes `.webm`. Convert to mp4:
  `ffmpeg -y -i in.webm -c:v libx264 -pix_fmt yuv420p -movflags +faststart out.mp4`
  If `ffmpeg` is missing, stop and tell the user to install it. Do not upload webm.
- Only `.mp4` is delivered. No webm, gif or screenshots as the result.
- File name: exactly the unit name plus `.mp4` (Qase: the Test Title; Jira task: `AC_01.mp4`, `EC_02.mp4`). Replace only characters Drive rejects (`/`).
- Keep the video short: no idle waiting; cut nothing, but do not record setup steps outside the case.

## Store
- Upload to the Drive folder named in the `Config` tab (key `VIDEO_FOLDER`), inside a subfolder named by the ticket key (`PYR-xxx`; create it if missing). If the same file name exists there, replace its content (same link), do not add a second file.
- Use the Google Drive connector. If an upload fails (size or error), say so and stop that item; never switch to another host.
- Do not change sharing settings on your own. If a viewer cannot open a link, tell the user.
- The link you report is the file's Drive link (`webViewLink`) to the `.mp4`, not the folder.

## Hygiene
- Videos must not show passwords, tokens or personal data that is not test data. If one does, delete the upload and re-record.
- Keep local temp files in the scratchpad; delete them after upload.
