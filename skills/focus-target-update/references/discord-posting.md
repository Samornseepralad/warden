# Discord posting and editing — recipes that work

Used by `/focus-target-update` step 6. Discord has no plugin here; everything goes through the logged-in Chrome tab. Follow the numbered steps exactly: the obvious shortcuts fail (details below).

**Posting.** Only after the user approved the exact text. **This is the only posting recipe that worked; the obvious ones fail** (cmux `type_text` never lands; JS `execCommand insertText` leaves Discord's editor out of sync so Return clears the box without sending; Return can even switch the tab to another channel):
1. Save the user's current clipboard into a private temp file (`CLIP=$(mktemp) && chmod 600 "$CLIP" && pbpaste > "$CLIP"`) and tell the user it will be replaced, then write the approved text to a file and `pbcopy < file`; check `pbpaste | wc -c`. Restore it (`pbcopy < "$CLIP"; rm -f "$CLIP"`) on **every** way out: after posting, after an abort, after an error.
2. Open a **dedicated new Chrome tab** on the target channel URL and use only that tab (a shared Discord tab got switched to other servers while working). Chrome pid + window id from cmux `list_windows`; `tell application "Google Chrome" to activate`.
3. Discord restores the last draft after load: read the box length (`[role=textbox]`, empty ≤ 2). If not empty, **tell the user first** what draft is in the box (it may be theirs), then clear with **real keys** (cmux `hotkey cmd+a`, `press_key delete`, `delivery_mode:"foreground"`, pid + window id) — never clear or insert with JS.
4. JS only to read and focus the box.
5. `hotkey cmd+v` (foreground). Read the box back: exactly one copy, header line first, `location.pathname` still the target channel.
6. **Right before sending re-check `location.pathname`; abort if it is not the target channel.** Then `press_key return` (foreground).
7. **Verify it posted:** after 3–4 s the box is empty AND a message with the header text exists in `li[id^=chat-messages-]` (and `screencapture -x -o -l <windowId>` shows it). Box still holds the text → press Return again (max 3 tries). Box empty but message not visible → do **not** resend: reload the channel and look again; resend only if a reload still shows nothing.
8. Post nowhere else. Restore the clipboard. Leave the dedicated tab open and tell the user its position. End the cmux session.

### Editing an already-posted message
Messages are edited, not deleted and reposted (keeps the timestamp). Same dedicated tab, Chrome frontmost.
1. Focus the empty main box (JS `.focus()`), `press_key up` (foreground) → the user's last message opens in edit mode ("escape to cancel • enter to save"; two `[role=textbox]` exist, the edit one is index 0).
1b. **Check you opened the right message**: the first line of the edit box must be the target-result header (`<first name> <d Mon>: …`) **with the date of the day being edited** (not just any earlier result). If it is not (something else was posted after it), press Escape and stop.
2. Clear the edit box with `hotkey cmd+shift+up` then `press_key delete` (`cmd+a` does not select there; plain `delete` removes one char). Edit box length ≤ 2.
3. **Never press Return while the edit box is empty** — Discord offers to delete the message.
4. `cmd+v` does not paste in the edit box. Dispatch a synthetic paste so Discord's own handler inserts the text: on `document.querySelectorAll('[role=textbox]')[0]`, `dispatchEvent(new ClipboardEvent('paste',{clipboardData:dt,bubbles:true,cancelable:true}))` with `dt=new DataTransfer(); dt.setData('text/plain', text)`. Build the JS string with Python `json.dumps` so Thai, emoji and newlines survive osascript. Read the box back.
5. `press_key return` (foreground); verify the new text and `(edited)`.
