# Package Tracker: DHL, FedEx, UPS, USPS & 17TRACK inside Claude

Track packages without leaving Claude. Paste one tracking number or a whole list, and the plugin returns the current status, estimated delivery, last scan and full checkpoint history for **DHL, FedEx, UPS and more than 1,000 other carriers through 17TRACK** (USPS, China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail, Australia Post and others).

It works in Claude chat, in Cowork and in Claude Code. There are no carrier developer accounts to apply for and no API keys to paste. Lookups run as pay-per-record Apify Actors on your own Apify account, and you are only charged for records that come back with data.

## What you can ask

- "Track 1Z999AA10123456784 and 794644790138."
- "Where is my package? The number is 7777777770."
- "Here are 60 tracking numbers from my Shopify export. Which ones are late?"
- "Track this AliExpress order: YT2534100012345678."
- "Give me the full scan history for this UPS package."
- "What will it cost to track 200 DHL numbers?"
- "Draft a reply to this customer whose FedEx parcel shows a delivery exception."

Claude detects the carrier from the number format, picks the right tracker, waits for the result and shows a table. Numbers the carrier cannot find are listed with the reason instead of being dropped.

## Carriers covered

| Carrier | Tracker used | Numbers per run | What you get |
|---|---|---|---|
| DHL Express, DHL Parcel, DHL eCommerce, DHL Global Forwarding | DHL Package Tracking Scraper | 10 | status, product, ETA, origin, destination, current location, full checkpoint list |
| FedEx (Ground, Express, Home Delivery, SmartPost, door tags) | FedEx Package Tracking Scraper | 30 | status, service, ship date, delivery date, origin, destination, scan events |
| UPS (1Z numbers, InfoNotice) | UPS Package Tracking Scraper | 25 | status, estimated delivery, last update, delivered-to and received-by |
| UPS full scan history, FedEx International Connect | Multi-Carrier Package Tracking | 25 | normalized status, origin, destination, every scan event |
| USPS, China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail, La Poste, Correos, PostNL, Australia Post, Canada Post, Japan Post, J&T, Aramex and 1,000+ more | 17TRACK Package Tracking Scraper | 10 | status, lifecycle category, carriers on the route, origin and destination country, ETA, checkpoints, in 9 languages |

Not supported: Amazon TBA numbers (Amazon does not expose them publicly), proof-of-delivery images and signatures, label creation, rates, pickups and claims.

## How to set it up

1. Install the plugin from the Claude directory (claude.ai > Customize > Plugins) or in Claude Code with `claude plugin install trackbuddy-parcel-tracker@imzlab` after adding the marketplace `afzalbuilds/trackbuddy-parcel-tracker`.
2. Open the plugin's **Connectors** tab and connect **Apify**. A browser window opens so you can sign in or create a free Apify account. Free accounts include monthly usage credit, so you can try the tracker without adding a card.
3. Ask Claude to track a number.

In Claude Code, `/mcp` shows the connection state and `/trackbuddy-parcel-tracker:track <numbers>` runs a lookup directly.

## Pricing

The plugin itself is free. Each tracked record is billed by Apify on your account at the Actor's pay-per-event price. Prices below are the Apify Store prices on the FREE tier as of 2026-11-03. Paid Apify plans get a discount, and blocked or not-found numbers are never charged as records.

| Tracker | Per tracked record | Per run start | 10 records cost |
|---|---|---|---|
| DHL | USD 0.020 | USD 0.00005 | USD 0.20 |
| FedEx | USD 0.020 | USD 0.005 | USD 0.205 |
| UPS | USD 0.030 | USD 0.005 | USD 0.305 |
| 17TRACK (USPS, China Post, YunExpress and others) | USD 0.039 | USD 0.005 | USD 0.395 |
| Multi-carrier (UPS scan history, FedEx International Connect) | USD 0.040 (from 2026-11-06) | USD 0.005 | USD 0.405 |

Before any batch over 10 numbers, Claude tells you the estimated cost and asks before running. Ask "what will it cost to track these" to get an estimate without running anything.

## Data and privacy

- Tracking numbers you give Claude are sent to Apify's hosted MCP server (`mcp.apify.com`), which runs the tracking Actor on your Apify account. The Actor queries the carrier's public tracking page.
- Results are stored in your Apify account's datasets for your plan's retention period, where you can download them as CSV, Excel or JSON. The plugin stores nothing itself and sends nothing to any other service.
- Tracking records can include delivery addresses and recipient names shown on the carrier's public page. Treat them as personal data.
- You authorize Apify through OAuth. The plugin never sees or stores your Apify token.
- Full policy: https://imzlab.com/privacy#package-tracker. Terms: https://imzlab.com/terms#package-tracker. Support: https://imzlab.com/support#package-tracker

## Frequently asked questions

**Do I need a FedEx, UPS or DHL developer account?** No. The trackers read the same public tracking pages you would open in a browser, and the plugin never asks for carrier API keys.

**How many numbers can I track at once?** Per run: 10 for DHL and 17TRACK, 25 for UPS and the multi-carrier tracker, 30 for FedEx. Claude splits longer lists into several runs automatically and merges the results.

**What happens with a number that is not found?** It costs nothing as a record and appears in the results table with the carrier's message, for example "no tracking information" or "label created, not yet scanned". New labels can take 24 to 48 hours to show scans.

**Can I track USPS?** Yes, through 17TRACK, which aggregates USPS. The UPS, FedEx and DHL trackers are carrier-specific.

**Can it track AliExpress, Temu or Shein orders?** Yes. Those parcels travel on carriers such as YunExpress, Cainiao, 4PX, Yanwen and China Post, all covered by 17TRACK. Use the tracking number from the order page, not the order number.

**Is this an official carrier tool?** No. This plugin and the Actors are independent and not affiliated with DHL, FedEx, UPS, USPS or 17TRACK. Status wording comes from the carriers.

**Can I schedule a daily check?** In Claude Code or Cowork, ask the tracking auditor agent to run your list and report exceptions. For unattended schedules, create an Apify schedule for the Actor in Apify Console.

## Skills and commands in this plugin

- **track-package**: single or mixed-carrier lookups with automatic carrier detection.
- **bulk-tracking-report**: paste a list or a CSV column, get a status table, exceptions first, plus a downloadable CSV.
- **delivery-exceptions**: triage delayed, failed, returned and held shipments and draft customer replies.
- **tracking-cost-estimate**: estimate the Apify cost of a batch before running it.
- **/trackbuddy-parcel-tracker:track** and **/trackbuddy-parcel-tracker:track-report** commands (Cowork and Claude Code).
- **tracking-auditor** agent (Cowork and Claude Code): runs a list and writes an exceptions report.

## Source and support

Source: https://github.com/afzalbuilds/trackbuddy-parcel-tracker. Issues and feature requests: https://github.com/afzalbuilds/trackbuddy-parcel-tracker/issues. The Actors are published on Apify Store by Imzlab Technologies under the `muhammadafzal` account.

Last verified: 2026-11-03.
