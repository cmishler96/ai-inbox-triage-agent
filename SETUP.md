# Demo 02 setup: what you do by hand

These steps need you, because they involve your Google account, an API key, or a consent screen.

## 1. Install clasp (one time)
Install Node.js LTS from nodejs.org, then:
```
npm install -g @google/clasp
clasp login
```
Also turn on the Apps Script API at https://script.google.com/home/usersettings. clasp can't push without it.

## 2. Create the project and push
From this folder:
```
clasp create --type standalone --title "Inbox Triage Demo"
clasp push
```
`clasp create` writes `.clasp.json` with your script ID. If you created the project in the browser
instead, copy `.clasp.json.example` to `.clasp.json` and paste the script ID from Project Settings.

## 3. Add the API key
In the Apps Script editor, open **Project Settings → Script Properties** and add `ANTHROPIC_API_KEY`.
Never put the key in code.

## 4. Authorize and set up
In the editor, run `setup()` and approve the consent screen. Because this is an unverified personal project,
you'll click **Advanced → Go to Inbox Triage Demo (unsafe)**. The script asks for full Gmail, Sheets, external
requests, triggers, and your email address. The log prints the sheet URL.

Optional: in the Settings tab, change **Owner name** from "Bob" to the name you want on drafts. If you change
**Digest hour**, run `setup()` again so the trigger moves.

## 5. Demo run
1. `seedSampleData()`: about 30 seconds for 15 emails. If it reports fewer than 15 found, run `fixSeedUnread()`.
2. Record the "before": unread inbox.
3. `triageInbox()`: run it by hand rather than waiting for the 15-minute trigger.
4. Record the "after": labels, Priority/High threads, drafts threaded under the emails, the Action Items tab, and
   the "Action required" injection email sitting harmlessly in FYI/Low.
5. `sendMorningDigest()` sends the digest right away.
6. `resetDemo()` gets you ready to record again.

## Notes
- The only email the agent ever sends is the digest, and only to you. Every reply is a draft.
- Every Claude call is in the **AI Log** tab with tokens and raw JSON.
