# n8n marketing automations

Four n8n workflows for the repetitive end of running a small brand's
marketing: sending the link to everyone who comments "GUIDE", sorting real
enquiries from spam, asking for reviews at the right moment, and getting
more than one post out of a good piece of writing.

I'm [Kaella](https://kaellastudio.com), and I do this work for small beauty,
baby and lifestyle brands. Everything here runs on made-up sample data. No
client data, client names or real credentials are in this repo.

| Workflow | What it does | Starts when |
|---|---|---|
| [Comment to DM](workflows/01-comment-to-dm.json) | Sends the link by DM when someone comments a keyword | Someone comments on Instagram |
| [Enquiry triage](workflows/02-enquiry-triage.json) | Sorts, routes and acknowledges website enquiries | A form is submitted |
| [Review request](workflows/03-review-request.json) | One email a week after delivery, asking how it's going | An order is delivered, then hourly |
| [Content repurposing](workflows/04-content-repurposing.json) | Turns a long post into captions for three platforms | Every 6 hours |

## A note before you use these

These are portfolio builds. I didn't have an Instagram account or a store of
my own to point them at, so they've been tested on a real n8n server with
Meta, Google, OpenAI and Slack replaced by local stand-ins (more on that
under [Testing](#testing)). To run them for real, swap these placeholders
for your own and do one test run before switching anything on:

| Placeholder | Where | Replace with |
|---|---|---|
| `SHEET_ID_HERE` | Every Google Sheets node | Your Google Sheet ID |
| `VERIFY_TOKEN_HERE` | *Verify token matches*, in Comment to DM | Any long random string, also typed into the Meta app dashboard |
| `#enquiries` | *Notify Slack*, in Enquiry triage | Your Slack channel |
| `example.com` links | Comment to DM keywords, Review request emails | Your real guide, pricing, review and unsubscribe pages |
| Sign-off and wording | *Acknowledge sender*, *Send review request* | Your own name and brand voice |
| Workflow timezone | *Workflow settings* | The brand's own timezone |
| Credentials | Every node that needs one | Your own. None are included |

## Why these four

Small brands rarely lose customers because their automation isn't clever
enough. They lose them because the founder is asleep when someone comments
"PRICES" at half eleven at night, and by morning that person has bought
somewhere else. Each of these covers one of those gaps.

## 01 · Comment to DM

Someone comments a keyword on a post. Within seconds they get the link in
their DMs, and a short public reply goes up under their comment so other
people can see it worked.

Comment bots usually go wrong in the same few ways, so most of this workflow
is about those:

- Your own replies come back through the same webhook. Without a check on
  who sent the comment, the bot replies to its own reply and keeps going,
  which is the quickest way to get an account restricted. The check uses
  the account ID that Meta includes in every event, so there's no setting to
  forget.
- Keywords match whole words only, so "guiding" doesn't trigger GUIDE. If two
  keywords match, the longer one wins.
- Each person gets one DM a day at most, checked against a sheet before
  sending. Meta restricts accounts that keep messaging the same person.
- The public "check your DMs" reply only goes up after the DM has actually
  been sent. If the DM fails, nothing is logged, so it doesn't use up that
  person's one DM for the day.
- Meta needs a 200 response within five seconds or it sends the event again,
  so the workflow answers straight away on a separate branch and does the
  work afterwards. Doing it the other way round is how you end up sending
  duplicate DMs.
- When you first register the webhook, Meta sends a verification request
  and won't send any comments until it gets the right answer back. A second
  webhook on the same path handles that, and turns away anyone without your
  verify token.

Worth knowing: Instagram only lets you privately reply to a comment once, and
only within seven days of it being posted. So this can't message anyone who
hasn't commented, and it isn't a way to broadcast to followers. If someone
sells you that, it breaks Meta's rules.

## 02 · Enquiry triage

A form submission gets classified, routed, acknowledged and logged before
you've opened your inbox.

- Bots are filtered out before the AI step, with a hidden honeypot field and
  an email format check, so spam never costs anything.
- The model runs at temperature 0, so the same enquiry is classified the
  same way every time.
- If the model is down, or replies with something that doesn't parse or
  isn't one of the expected values, the enquiry is marked `unclear` and still
  comes to you. Losing a real lead to a parsing error would be the worst
  outcome, so the fallback errs towards sending things to a person.
- Every enquiry is logged exactly once, the binned ones included, before the
  email or Slack step has a chance to fail.
- The automatic reply promises an answer within one working day, not an
  hour. I'd rather the automation promise something a person can keep.

## 03 · Review request

When an order is delivered, the workflow waits seven days and sends one
email asking how they're getting on. It never sends a second.

- Carriers send the same delivery event more than once, sometimes days
  apart. If an order is already in the queue it's left alone, so a late
  duplicate can't reset a sent row back to pending or restart the wait. Two
  copies arriving together become one row.
- The queue is checked every hour rather than once a day, so each customer
  gets the email during their own daytime. 10am in London is 8pm in Sydney.
  An unrecognised timezone counts as sendable, so the row doesn't sit there
  forever.
- There are four separate reasons not to send, checked one at a time: it's
  already been sent, the address is on the suppression list, the seven days
  aren't up, or it's the wrong time of day where they are. Getting any one
  of those wrong is a real complaint.
- An order with any row already marked sent is skipped, so a stray duplicate
  row in the sheet doesn't get emailed every hour.
- The email offers to fix a problem before it asks for a review, and has a
  one-click opt-out. Asking an unhappy customer for a public review is a
  reliable way to get a one-star.
- A row is only marked sent once Gmail confirms the email went. A failed send
  stays pending, gets retried the next hour, and doesn't hold up the rest of
  the batch. If the sheet update itself fails after sending, that row could
  go out again on the next run, so it's worth keeping an eye on failed
  executions.

## 04 · Content repurposing

A long-form post is turned into an Instagram caption, a LinkedIn post and a
TikTok hook, written back to the same sheet.

- Rows that are already done are skipped. Without that check it would
  reprocess the whole sheet every six hours, and the bill would creep up
  without anyone noticing. Rows marked `needs review` are skipped too, so
  they wait for a person instead of being re-billed each run. Clear the
  status to send one through again. Anything under 200 characters never
  reaches the model.
- It handles five rows per run at most, so a 300-row backlog doesn't fire
  300 calls at once.
- Character limits are enforced in code afterwards, cutting at a word
  boundary, rather than trusting the prompt. The model does go over, and a
  caption the platform rejects at publish time is easy to miss until the
  slot has passed.
- If the model fails and a platform comes back blank, the row is marked
  `needs review`. An empty caption never gets written into the calendar as
  if it were finished.
- The prompt tells the model not to add claims that weren't in the original:
  no statistics, testimonials or results it made up.

## Setting them up

You'll need an n8n instance (cloud or self-hosted) on a recent version.

1. Import a workflow: *Workflows → Import from File*, then pick the JSON
   file.
2. Add your credentials. Every node that needs one is left empty on purpose.
3. Replace the placeholders listed above. For `VERIFY_TOKEN_HERE` in Comment
   to DM, pick any long random string. Put the same string into the Meta
   app dashboard as the verify token, with the workflow's production webhook
   URL (path `ig-comment`) as the callback URL. Activate the workflow before
   you press Verify. The unsubscribe link in Review request needs something
   on your side that adds the address to the `suppression` tab.
4. Create the sheet tabs, with these exact names and header rows:

   | Tab | Used by | Columns |
   |---|---|---|
   | `sent-log` | 01 | `fromId`, `username`, `matched`, `commentId`, `sentAt` |
   | `enquiries` | 02 | `submittedAt`, `name`, `email`, `brand`, `service`, `fit`, `urgency`, `reason` |
   | `review-queue` | 03 | `orderId`, `email`, `firstName`, `product`, `timezone`, `deliveredAt`, `sendAfter`, `status`, `sentAt` |
   | `suppression` | 03 | `email` |
   | `long-form` | 04 | `rowId`, `title`, `body`, `status`, `instagram`, `linkedin`, `tiktok`, `hashtags`, `generatedAt` |

   The workflows write in raw cell format, so IDs and dates stay as plain
   text instead of being turned into numbers or local date formats.
5. Set each workflow's timezone (*Workflow settings → Timezone*) to the
   brand's own. "One DM per person per day" means a day in that timezone.
6. Test on pinned data before you activate anything. Three of these send
   real messages to real people as soon as they run. The
   [sample data notes](sample-data/) say which file goes on which node.

## What isn't here, on purpose

No credentials of any kind: no `.env` files, keys or tokens. Nothing scraped
from behind a login. Instagram and Faire both forbid that in their terms,
whatever a tutorial says. And no client work. These are patterns I've
rebuilt from scratch against invented data.

## Testing

All four workflows have been run end to end on n8n 2.45.7, using the sample
data and the awkward cases around it: repeats, duplicates, a model outage, a
model reply that isn't JSON, a failed email halfway through a batch, an
expired comment, and an empty suppression list. Comment to DM was also run
as an active workflow over real HTTP, including Meta's verification request.
Every node's settings have been checked against n8n's own node definitions
for that version.

In the first pass, Google Sheets, Gmail, Slack, OpenAI and the Meta Graph API
were stand-ins built from the real nodes' definitions, so n8n evaluated every
expression, but nothing reached Google, Slack, OpenAI or Meta.

A second pass ran all four as published workflows on a live n8n 2.45.7
server, with those services answered by a local stand-in over HTTPS, so the
real nodes made their real requests. That found two more problems, both now
fixed:

- Every Google Sheets write failed after import. From version 4.4 the Sheets
  node needs a column schema next to the mapping. The n8n editor fills it in
  when you open a node, so an imported workflow without one fails on its
  first write. Each write node now includes its schema.
- A duplicate row in the review queue was being emailed every hour, because
  marking a row sent only updates the first match. The hourly check now
  skips any order that already has a sent row.

What hasn't been done yet is a run against live accounts. That's the last
check for each workflow, and I'll update this section when it's done.

## Licence

MIT. Use them, change them, ship them. Credit is welcome but not required.
