# n8n marketing automations

Working n8n workflows for the unglamorous parts of running a small brand's
marketing: answering the same DM for the hundredth time, triaging enquiries,
and chasing the follow-ups that quietly never happen.

Built by [Kaella Studio](https://kaellastudio.com). Every workflow here is
importable, commented, and runs on synthetic data — **no client data, no
client names, no real credentials**.

---

## Why these three

Small brands do not lose customers because their automation is unsophisticated.
They lose them because the founder is asleep when someone comments "PRICES" at
half eleven at night, and by morning that person has bought elsewhere.

So each workflow closes one gap where a slow human response costs money.

| Workflow | Closes | Trigger |
|---|---|---|
| [Comment to DM](workflows/01-comment-to-dm.json) | Comment at midnight, reply at nine | Instagram comment webhook |
| [Enquiry triage](workflows/02-enquiry-triage.json) | Good leads buried under spam | Form submission |
| Review request follow-up | Nobody asks, so nobody reviews | *in progress* |

---

## 01 · Comment to DM

Someone comments a keyword on a post. They get the link in their DMs within
seconds, plus a public reply under their comment so everyone else scrolling the
thread sees it worked.

**The parts that matter, and are usually missing:**

- **It does not answer itself.** Your own replies come back through the same
  webhook. Without a check on the sender ID, the workflow replies to its own
  reply, and keeps going. This is the single most common way a comment bot
  gets an account restricted.
- **Whole-word keyword matching.** `\bGUIDE\b`, so a comment saying "guiding"
  does not trigger the GUIDE rule. Longest keyword wins when two match.
- **One DM per person per day**, checked against a sheet before sending. Meta
  restricts accounts that message the same user repeatedly.
- **The webhook acknowledges Meta immediately** on a separate branch. Meta
  requires a 200 within five seconds and retries if it does not get one —
  which means duplicate DMs if you do the work first and respond after.

**Honest limits.** Instagram Private Replies work **once per comment**, and
only within **seven days** of it. This cannot DM someone who has never
commented, and it is not a broadcast tool. Anyone selling you that is
describing a policy violation.

## 02 · Enquiry triage

A form submission gets classified, routed, acknowledged and logged before you
have looked at your inbox.

- **Bots die before they cost money.** A honeypot field and an email format
  check run *before* the LLM call, so spam never becomes a bill.
- **`temperature: 0`.** The same enquiry classifies the same way twice.
  Triage that changes its mind is not triage.
- **Failure routes to a human.** If the model is down or returns
  unparseable JSON, the enquiry is marked `unclear` and still reaches you.
  Losing a real lead to a JSON parse error is the worst possible outcome, so
  the fallback is deliberately generous rather than clever.
- **The auto-reply promises one working day**, not one hour. An automation
  that writes cheques the human cannot cash is worse than no automation.

---

## Running these

Requires an n8n instance (self-hosted or cloud) on a recent version.

1. **Import.** In n8n: *Workflows → Import from File →* pick a JSON file.
2. **Add credentials.** Every node needing auth is left unconfigured on
   purpose. Nothing here contains a key, and nothing should.
3. **Replace the placeholders.** Search each workflow for:
   - `SHEET_ID_HERE` — your Google Sheet ID
   - `IG_BUSINESS_ID` — set as an n8n environment variable, not pasted inline
   - `#enquiries` — your Slack channel
4. **Test on pinned data first.** Both workflows send real messages to real
   people the moment they fire. Pin sample data onto the trigger node and run
   it manually before you activate anything.

Sample payloads matching each trigger are in [`sample-data/`](sample-data/).

## What is deliberately not here

- **Credentials of any kind.** No `.env`, no keys, no tokens.
- **Anything scraped from a login-walled platform.** Instagram and Faire both
  prohibit it in their terms, whatever a tutorial told you.
- **Client work.** These are rebuilds of patterns, written from scratch
  against synthetic data.

## Status

Workflows 01 and 02 are complete and structurally validated (JSON parses,
every connection resolves to a real node, no duplicate node names). They have
**not yet been run end to end on a live n8n instance** — that pass is next,
and this line gets deleted when it is done.

## Licence

MIT. Use them, change them, ship them. Attribution welcome, not required.
