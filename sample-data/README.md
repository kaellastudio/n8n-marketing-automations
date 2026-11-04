# Sample data

Made-up inputs for trying the workflows without connecting a real account.
Every name, email, brand and ID is invented, and the domains use the reserved
`.test` ending, so nothing can be delivered anywhere by accident.

To use one, open the workflow in n8n, click the node named in the table, choose
*Edit Output* (pinned data), paste in the file and run the workflow by hand.
Each file is a list, and every entry becomes one item. The `_case` and
`_comment` fields are only there to explain the entry, and the workflows ignore
them. Nothing leaves n8n until you connect real credentials.

| File | Pin onto | What's in it |
|---|---|---|
| `01-comment-webhook.json` | *Meta webhook (comment)*, in Comment to DM | One real keyword comment, an echo of the brand's own reply (should be ignored), and "guiding", which shouldn't trigger GUIDE |
| `02-enquiry-submissions.json` | *Enquiry form*, in Enquiry triage | A good lead, an agency pitch, a bot that filled in the hidden field, and a vague message that should still reach a person |
| `03-delivered-orders.json` | *Order delivered webhook*, in Review request | A duplicate delivery event (should end up as one row), a parcel still in transit, a broken email address, and a customer in Sydney |
| `03-review-queue-rows.json` | *Read queue*, in Review request. Also pin `[{"email": "unsubscribed@example.test"}]` onto *Read suppression list* | One row due in London and one in Sydney (only whichever is between 9:00 and 18:59 locally goes out), plus rows already sent, still waiting, unsubscribed, and with a timezone that doesn't exist |
| `04-long-form-rows.json` | *Read long-form sheet*, in Content repurposing | One row ready to go, one already done, and one too short to be worth a model call. Only the first should reach the model |

Most of these are there for the cases that go wrong, not the ones that go
right. That's where these workflows usually fail.
