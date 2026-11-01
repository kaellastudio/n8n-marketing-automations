# Sample data

Synthetic payloads for testing the workflows without connecting a live
account. Every name, email, brand and ID here is invented. Domains use the
reserved `.test` TLD so nothing can accidentally resolve or receive mail.

**Use them like this:** open a workflow in n8n, click the node named in the
table below, *Edit Output* / *Pin Data*, paste the matching file, then run the
workflow manually. Each file is a list, and each entry becomes one item. The
`_case` and `_comment` fields only explain the entry; the workflows ignore
them. Nothing leaves n8n until you connect real credentials.

| File | Pin onto | Deliberately includes |
|---|---|---|
| `01-comment-webhook.json` | Comment to DM: *Meta webhook (comment)* | A self-echo that must be dropped, and "guiding" which must NOT match "GUIDE" |
| `02-enquiry-submissions.json` | Enquiry triage: *Enquiry form* | An agency pitch, a tripped honeypot, and a genuinely ambiguous message |
| `03-delivered-orders.json` | Review request: *Order delivered webhook* | A duplicate delivery event (must end up as one row), a parcel still in transit, a malformed email, and a Sydney customer whose civil hours differ from London's |
| `03-review-queue-rows.json` | Review request: *Read queue* (and pin `[{"email": "unsubscribed@example.test"}]` onto *Read suppression list*) | One row due in London and one in Sydney (only the one inside its own 9:00 to 18:59 goes out), plus rows already sent, still waiting, unsubscribed, and with an unrecognised timezone |
| `04-long-form-rows.json` | Content repurposing: *Read long-form sheet* | A row already marked done, and one too short to be worth a model call. Neither should reach the model |

The awkward cases are the point. A workflow that handles the happy path is
not finished.
