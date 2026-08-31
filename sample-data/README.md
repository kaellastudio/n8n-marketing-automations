# Sample data

Synthetic payloads for testing the workflows without connecting a live
account. Every name, email, brand and ID here is invented. Domains use the
reserved `.test` TLD so nothing can accidentally resolve or receive mail.

**Use them like this:** open a workflow in n8n, click the trigger node,
*Edit Output* / *Pin Data*, paste the matching file, then run the workflow
manually. Nothing leaves n8n until you connect real credentials.

| File | Feeds | Deliberately includes |
|---|---|---|
| `01-comment-webhook.json` | Comment to DM | A self-echo that must be dropped, and "guiding" which must NOT match "GUIDE" |
| `02-enquiry-submissions.json` | Enquiry triage | An agency pitch, a tripped honeypot, and a genuinely ambiguous message |
| `03-delivered-orders.json` | Review request | A duplicate delivery event, a parcel still in transit, and a Sydney customer whose civil hours differ from London's |
| `04-long-form-rows.json` | Content repurposing | A row already marked done, and one too short to be worth a model call |

The awkward cases are the point. A workflow that handles the happy path is
not finished.
