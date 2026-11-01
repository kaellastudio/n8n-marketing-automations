# n8n marketing automations

Working n8n workflows for the unglamorous parts of running a small brand's
marketing: answering the same DM for the hundredth time, triaging enquiries,
and chasing the follow-ups that quietly never happen.

Built by [Kaella Studio](https://kaellastudio.com). Every workflow here is
importable, commented, and runs on synthetic data. **No client data, no
client names, no real credentials.**

> **Portfolio build.** These were built to show the approach, not run on a
> live brand. There were no Instagram or store pages of my own to point them
> at, so they were tested on a real n8n instance with Meta, Google, OpenAI
> and Slack replaced by local stand-ins (details under [Status](#status)).
> To use them for real, replace the placeholders below and connect your own
> accounts, then do one test run before switching anything on:
>
> | Placeholder | Where | Replace with |
> |---|---|---|
> | `SHEET_ID_HERE` | Every Google Sheets node | Your Google Sheet ID |
> | `VERIFY_TOKEN_HERE` | *Verify token matches*, Comment to DM | Any long random string, also typed into the Meta app dashboard |
> | `#enquiries` | *Notify Slack*, Enquiry triage | Your Slack channel |
> | `example.com` links | Comment to DM keywords, Review request emails | Your real guide, pricing, review and unsubscribe pages |
> | Sign-off and wording | *Acknowledge sender*, *Send review request* | Your own name and brand voice |
> | Workflow timezone | *Workflow settings* | The brand's own timezone |
> | Credentials | Every node that needs one | Your own. None are included |

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
| [Review request](workflows/03-review-request.json) | Nobody asks, so nobody reviews | Delivery webhook + hourly sweep |
| [Content repurposing](workflows/04-content-repurposing.json) | One good post used once | Every 6 hours |

---

## 01 · Comment to DM

Someone comments a keyword on a post. They get the link in their DMs within
seconds, plus a public reply under their comment so everyone else scrolling the
thread sees it worked.

**The parts that matter, and are usually missing:**

- **It does not answer itself.** Your own replies come back through the same
  webhook. Without a check on the sender ID, the workflow replies to its own
  reply, and keeps going. This is the single most common way a comment bot
  gets an account restricted. The check compares against the account ID Meta
  sends in every event, so there is no environment variable to forget.
- **Whole-word keyword matching.** `\bGUIDE\b`, so a comment saying "guiding"
  does not trigger the GUIDE rule. Longest keyword wins when two match.
- **One DM per person per day**, checked against a sheet before sending. Meta
  restricts accounts that message the same user repeatedly.
- **The public reply only goes up once the DM has gone out.** A "check your
  DMs" over an empty inbox is worse than no reply. A failed DM is not logged,
  so it does not use up that person's one DM for the day.
- **The webhook acknowledges Meta immediately** on a separate branch. Meta
  requires a 200 within five seconds and retries if it does not get one,
  which means duplicate DMs if you do the work first and respond after.
- **It answers Meta's verification call.** When you register the callback
  URL, Meta sends a GET with a challenge and sends nothing else until it gets
  that challenge back. A second webhook on the same path handles it, and
  refuses anyone without your verify token.

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
- **Failure routes to a human.** If the model is down, or returns JSON that
  does not parse or does not match the expected values, the enquiry is marked
  `unclear` and still reaches you.
  Losing a real lead to a JSON parse error is the worst possible outcome, so
  the fallback is deliberately generous rather than clever.
- **Every enquiry is logged once**, spam-binned ones included, before the
  email or Slack step has a chance to fail.
- **The auto-reply promises one working day**, not one hour. An automation
  that writes cheques the human cannot cash is worse than no automation.

## 03 · Review request

An order is delivered, seven days pass, and the customer gets one email asking
how they are getting on. Never twice.

- **Duplicate delivery events are expected.** Carriers re-send them, sometimes
  days apart. An order already in the queue is left alone, so a late duplicate
  cannot reset a sent row to pending or restart the wait, and two copies in
  the same batch become one row.
- **The sweep runs hourly, not daily**, so each recipient can be sent inside
  *their* civil hours. A 10am London send is 8pm in Sydney. The check reads the
  customer's own timezone, and an unrecognised one is treated as sendable
  rather than stranding that row forever.
- **Four separate reasons not to send**, checked one at a time rather than
  folded into one clever condition: already sent, on the suppression list,
  still inside the wait, or the wrong hour where they are. Each one is a real
  complaint if you get it wrong.
- **The email offers a fix before it asks for a review**, and carries a
  one-click opt out. Asking an unhappy customer for a public review is how you
  buy a one-star.
- **Only rows Gmail confirms are marked sent**, straight after the send step.
  A failed send stays pending and is retried next hour without stopping the
  rest of the batch. If the sheet write itself fails after sending, those rows
  can go out again on the next sweep, so keep an eye on failed executions.

## 04 · Content repurposing

One long-form post becomes an Instagram caption, a LinkedIn post and a TikTok
hook, written back to the same sheet.

- **Processed rows are skipped.** Without a status check this reprocesses the
  entire sheet every six hours, and that is a bill that grows quietly until
  someone notices. Rows marked `needs review` are skipped too, so they wait
  for a human instead of being re-billed every run; clear the status to send
  one back through. Rows under 200 characters never reach the model either.
- **Five rows per run, maximum.** A 300-row backlog should not fire 300 calls
  in one go.
- **Character limits are enforced in code, not requested in the prompt.** The
  model will exceed them. A caption rejected by the platform at publish time is
  a failure nobody notices until the slot is missed, so limits are applied
  after the fact, truncating on a word boundary.
- **A blank platform is flagged, not shipped.** If the model fails, the row is
  marked `needs review` instead of writing an empty caption into the calendar
  and calling it done.
- **The prompt forbids invented claims.** No statistics, no testimonials, no
  results that were not in the source.

---

## Running these

Requires an n8n instance (self-hosted or cloud) on a recent version.

1. **Import.** In n8n: *Workflows → Import from File →* pick a JSON file.
2. **Add credentials.** Every node needing auth is left unconfigured on
   purpose. Nothing here contains a key, and nothing should.
3. **Replace the placeholders.** Search each workflow for:
   - `SHEET_ID_HERE`, your Google Sheet ID
   - `VERIFY_TOKEN_HERE`, in Comment to DM: any long random string. Type the
     same string into the Meta app dashboard as the verify token, with the
     workflow's production webhook URL (path `ig-comment`) as the callback URL.
     Activate the workflow before you click Verify.
   - `#enquiries`, your Slack channel
   - `example.com`, the keyword links in Comment to DM, and the review and
     unsubscribe links in Review request. The unsubscribe link needs something
     on your side that adds the address to the `suppression` tab.
4. **Create the sheet tabs.** Header row first, exact names:

   | Tab | Used by | Columns |
   |---|---|---|
   | `sent-log` | 01 | `fromId`, `username`, `matched`, `commentId`, `sentAt` |
   | `enquiries` | 02 | `submittedAt`, `name`, `email`, `brand`, `service`, `fit`, `urgency`, `reason` |
   | `review-queue` | 03 | `orderId`, `email`, `firstName`, `product`, `timezone`, `deliveredAt`, `sendAfter`, `status`, `sentAt` |
   | `suppression` | 03 | `email` |
   | `long-form` | 04 | `rowId`, `title`, `body`, `status`, `instagram`, `linkedin`, `tiktok`, `hashtags`, `generatedAt` |

   Writes use raw cell format, so IDs and dates stay as the text the workflow
   compares against, rather than being turned into numbers or locale dates.
5. **Set each workflow's timezone** (*Workflow settings → Timezone*) to the
   brand's own. "One DM per person per day" in Comment to DM means a day in
   that timezone.
6. **Test on pinned data first.** Three of these send real messages to real
   people the moment they fire. Pin the sample data (see
   [`sample-data/`](sample-data/) for which node takes which file) and run
   it manually before you activate anything.

Sample payloads matching each trigger are in [`sample-data/`](sample-data/).

## What is deliberately not here

- **Credentials of any kind.** No `.env`, no keys, no tokens.
- **Anything scraped from a login-walled platform.** Instagram and Faire both
  prohibit it in their terms, whatever a tutorial told you.
- **Client work.** These are rebuilds of patterns, written from scratch
  against synthetic data.

## Status

All four workflows have been run end to end on n8n 2.45.7 against the sample
data and the awkward cases around it: repeats, duplicates, a model outage, a
model reply that is not JSON, a failed email in the middle of a batch, an
expired comment, an empty suppression list. Comment to DM was also run as an
active workflow over real HTTP, including Meta's verification call. Every
node parameter has been checked against the n8n node definitions for that
version.

In those runs, Google Sheets, Gmail, Slack, OpenAI and the Meta Graph API
were stand-ins built on the real nodes' parameter definitions, so every
expression was evaluated by n8n itself, but no request reached Google, Slack,
OpenAI or Meta.

A second pass ran all four as published workflows on a live n8n 2.45.7
server, with those services answered by a local stand-in over HTTPS, so the
real Google Sheets, Gmail, Slack, OpenAI and Graph API nodes made their real
requests. That pass found two more problems, both fixed:

- **Every Google Sheets write failed on import.** Sheets nodes from v4.4 on
  need a column schema alongside the mapping. The n8n editor fills it in
  when a node is opened, so an imported workflow without it fails its first
  write. Each write node now carries its schema.
- **A duplicate row in the review queue was emailed every hour.** Mark sent
  only updates the first matching row, so the duplicate stayed pending. The
  sweep now skips any order that already has a row marked sent.

**They have not yet been run against live accounts.** The
first live run of each is the remaining check, and this paragraph gets
updated when it is done.

## Licence

MIT. Use them, change them, ship them. Attribution welcome, not required.
