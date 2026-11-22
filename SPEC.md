# Demo 02 — Inbox Triage Agent

## Pitch
"Every new email gets sorted, prioritized, and summarized. Routine replies are drafted for you,
action items land in a tracker, and you get one morning digest instead of 80 unread emails."

## Demo story
1. Inbox has ~15 unread emails: a billing question, a meeting request, a vendor quote, a
   complaint, newsletters, an urgent facilities issue, a sales lead, an internal FYI.
2. Agent runs. Each thread gets a `Triage/<Category>` label and a `Priority/<High|Normal|Low>` label.
3. Threads that need a reply get a Gmail **draft** (never sent) in the right tone.
4. `Action Items` sheet fills with tasks pulled from emails (owner, due date if stated, link).
5. Morning digest email: High-priority items first, one-line summary each, drafts waiting.

## Architecture
- Trigger: every 15 min → `triageInbox()`; daily 7:00 AM → `sendMorningDigest()`
- Source: `in:inbox is:unread -label:Triage` (newest 25 per run)
- AI: Claude classifies and drafts in one call per thread (thread text, max ~8k chars, newest last)
- Model: `claude-haiku-5-5` for classification (cheap/fast); optional upgrade to Sonnet for drafts

## Categories (configurable in Settings tab)
Billing · Scheduling · Support/Complaint · Sales Lead · Vendor · Internal · Facilities/Urgent · Newsletter/Promo · FYI

## AI output schema
```json
{
  "category": "one of the categories",
  "priority": "High | Normal | Low",
  "summary": "one sentence, <= 25 words",
  "needs_reply": true,
  "draft_reply": "plain text reply or null",
  "action_items": [{"task": "string", "due": "YYYY-MM-DD or null"}],
  "confidence": 0.0
}
```

## Rules
- Drafts only. The agent never calls `send` on anything except the digest to the owner.
- Newsletter/Promo: label only, no draft, no action items; optionally archive (Setting, default off).
- Don't mark as read (owner still sees it), except Newsletter/Promo if archive is on.
- Draft replies sign off with the owner name from Settings and never invent facts, prices, or dates;
  unknowns become `[CONFIRM: ...]` placeholders.
- Skip threads where the last message is from the owner.

## Sheet: `Inbox Triage Demo`
**Triage Log** — Processed At · From · Subject · Category · Priority · Summary · Draft Created (Y/N) · Thread Link
**Action Items** — Created · Task · Due · Source Subject · Thread Link · Done (checkbox)
**Settings** — Owner name · Categories (comma list) · Archive promos (TRUE/FALSE) · Digest hour
**AI Log** — same as other demos

## Acceptance criteria
- [ ] `setup()` creates labels, sheet tabs, both triggers
- [ ] `seedSampleData()` sends 15 realistic fake emails to the owner
- [ ] Re-running never re-triages a thread (Triage label = processed marker)
- [ ] Drafts appear threaded as replies in Gmail
- [ ] Digest groups by priority and links to each thread
- [ ] Prompt-injection guard: email text is wrapped as data; instructions inside emails are ignored
