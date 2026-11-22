# Changelog — Demo 02 Inbox Triage

## 2026-11-19 — Initial release

### Features
- `setup()` creates (or reopens) the `Inbox Triage Demo` sheet, its four tabs with headers, and the default
  Settings. It also creates every Gmail label and replaces both triggers (every 15 min `triageInbox`, daily
  `sendMorningDigest` at the Settings digest hour). You can run it again safely: it keeps your Settings edits and never
  duplicates triggers.
- `getSettings_()` reads the Settings tab (Owner name, Categories, Archive promos, Digest hour) and falls back
  to defaults.
- `logTriage_`, `logActionItems_` (with Done checkboxes), and `logAi_` (AI Log, raw reply truncated to 45k chars)
  write to the sheet.
- `sendMorningDigest()` covers the last 24h. It groups High → Normal → Low and links each thread. It marks threads that
  still have a draft, lists open action items, and counts filed promos. It's skipped if nothing was triaged.
- Failed threads get `Triage` + `Triage/Error` so they aren't retried forever, plus an ERROR row in Triage Log.

### Claude API
- Uses API version `2026-10-21` and **structured outputs** (`output_config.format` with a JSON schema).
  Category and priority are enums, so the reply always parses. Code still normalizes it defensively.
- `effort: "low"` on `claude-haiku-5-5`, which suits classification.
- Retries 429/529/5xx and network errors (exponential backoff, max 3). Treats `refusal` / `max_tokens` stop
  reasons as errors.
- The prompt now includes today's date and timezone, so "by Friday" becomes a real due date.
- Prompt-injection hardening: `<thread>` tags inside email text are neutralized, and the model is told
  that emails which try to instruct an AI go to FYI/Low with no reply.
- `[demo]` / `[demo-seed]` markers are stripped before Claude sees the text, so drafts read naturally.

### Sample data
- The 15 emails now use dates relative to the seed day, so the demo never looks stale. Fixed the receipts email,
  which listed a November date for October books.
- Drafts use the owner name from Settings.
- After sending, seeded threads are moved to Inbox and marked unread, because self-sent mail often arrives
  read. `fixSeedUnread()` re-runs that step if Gmail indexing lags.

### Added
- `retryErrors()` clears `Triage/Error` so the next run retries those threads.
- `resetDemo()` gets ready for a re-recording. It trashes only seeded demo threads (checked by subject
  prefix and body marker), deletes the agent's drafts on them, and clears the log tabs.
- Time guard: a run stops after 4.5 min, and the rest is picked up on the next trigger.
- The digest's own subject is excluded from the triage query.
- clasp: `.clasp.json.example` and `.claspignore`, which pushes only `*.gs` and `appsscript.json`.

### Scopes (`appsscript.json`)
- Replaced the three granular Gmail scopes with `https://mail.google.com/`. GmailApp requires it.
- Added `userinfo.email`, which `Session.getEffectiveUser().getEmail()` requires.

### Design decisions
- Category labels with a slash are flattened: `Triage/Support-Complaint`, not `Triage/Support/Complaint`.
  Otherwise Gmail nests them three levels deep.
- Thread links use `#all/<id>`, not `#inbox/<id>`, so they still work after a promo is archived.
