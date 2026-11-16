# n8n workflow templates: Package Tracker

Two importable n8n workflows that run the same Apify Actors as this plugin. Built with the official n8n skills plugin and n8n's instance-level MCP, on n8n Cloud with the Apify node (`@apify/n8n-nodes-apify` 0.9.0).

| File | What it does | Trigger | Needs |
|---|---|---|---|
| `track-package-form.json` | A public form takes one tracking number and shows the carrier, status and latest scan on the form's end page | n8n Form | Apify |
| `bulk-tracking-google-sheets.json` | Every morning reads tracking numbers from a Google Sheet, checks them 10 at a time, writes each status back and emails a digest of delivery exceptions | Schedule, daily 08:00 | Apify, Google Sheets, Gmail |

Both use the Actor `muhammadafzal/17track-package-tracking-scraper` (USPS and 1,000+ postal and cross-border carriers).

## Import

1. In n8n, install the verified community node **Apify** (Settings → Community nodes, or search "Apify" in the node panel).
2. Create a workflow, open the **⋯** menu and choose **Import from file**, then pick a JSON file from this folder.
3. Open each node marked with a warning and select your credential: an Apify account for the Apify nodes, plus Google Sheets and Gmail for the bulk workflow.
4. Publish the workflow.

## Sheet layout for the bulk workflow

One spreadsheet with a tab named `Shipments` and this header row:

`Tracking number` | `Status` | `Status detail` | `Carrier` | `Latest update` | `Location` | `Last checked`

Fill in `Tracking number` only. Rows whose `Status` is `delivered` are skipped on later runs. Set the alert address in the **Email exception digest** node.

## Cost

About USD 0.039 per package found plus USD 0.005 per run, billed by Apify to your own account. Numbers that are not found are not charged. The form workflow caps one run at USD 0.10 and the bulk workflow caps each batch of 10 at USD 0.50.

## What was tested

Each workflow passed n8n's validator and a test run with sample data in place of the Apify, Google Sheets and Gmail calls. The form's end page was opened and read. A live run with real accounts was not done, so check the first real run yourself.
