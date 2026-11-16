# Package Tracker: DHL, FedEx, UPS, USPS & 17TRACK inside Claude

Track packages without leaving Claude. Paste one tracking number or a whole list, and the plugin returns the current status, estimated delivery, last scan and full checkpoint history for **DHL, FedEx, UPS, Amazon Shipping, GOFO, SPX Express and more than 1,000 other carriers through 17TRACK** (USPS, China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail, Australia Post and others). It also tracks **Maersk ocean containers** by container, bill of lading or booking number.

It works in Claude chat, in Cowork and in Claude Code, and the same repository also installs as a native plugin in OpenAI Codex, Gemini CLI, GitHub Copilot, Grok, Cursor, Hermes Agent and OpenCode, with the skills usable in many more agents (see [Other AI agents](#other-ai-agents-codex-gemini-cli-copilot-and-more)). There are no carrier developer accounts to apply for and no API keys to paste. Lookups run as pay-per-record Apify Actors on your own Apify account, and you are only charged for records that come back with data.

## What you can ask

- "Track 1Z999AA10123456784 and 794644790138."
- "Where is my package? The number is 7777777770."
- "Here are 60 tracking numbers from my Shopify export. Which ones are late?"
- "Track this AliExpress order: YT2534100012345678."
- "Track TBA335250058574, it shipped with Amazon from a Shopify store."
- "Check these 300 SPX numbers from my Shopee shop and list the failed deliveries."
- "Where is container MSKU0078257 and when does it arrive?"
- "Give me the full scan history for this FedEx package."
- "What will it cost to track 200 DHL numbers?"
- "Draft a reply to this customer whose FedEx parcel shows a delivery exception."

Claude detects the carrier from the number format, picks the right tracker, waits for the result and shows a table. Numbers the carrier cannot find are listed with the reason instead of being dropped.

## Carriers covered

| Carrier | Tracker used | Numbers per run | What you get |
|---|---|---|---|
| DHL Express, DHL Parcel, DHL eCommerce, DHL Global Forwarding | DHL Package Tracking Scraper | 10 | status, product, ETA, origin, destination, current location, full checkpoint list |
| FedEx (Ground, Express, Home Delivery, SmartPost, door tags) | FedEx Package Tracking Scraper | 30 | status, service, ship date, delivery date, origin, destination, scan events |
| UPS (1Z numbers, InfoNotice) | UPS Package Tracking Scraper | 25 | status, estimated delivery, last update, delivered-to and received-by |
| FedEx International Connect (`KOT…`, `PRO…`), USPS direct from usps.com as a fallback | Multi-Carrier Package Tracking | 25 | normalized status, origin, destination, scan events |
| Amazon Shipping and Amazon Multi-Channel Fulfillment (TBA, TBC, TBM, UK numbers) | Amazon Shipping Tracking Scraper | 200 | status, expected and actual delivery time, shipper, last-mile carrier, last location, scan history (45 days) |
| GOFO and GOFO Express (formerly Cirro) | GOFO Tracking Scraper | 1,000 | status, delivery time, estimated delivery, transit days, last scan, full scan history |
| SPX Express / Shopee Express (Philippines, Malaysia, Indonesia, Vietnam, Singapore, Brazil) | SPX Tracking Scraper | 500 | status, delivery time, transit days, failed-attempt count and reason, hub events in English |
| Maersk containers, bills of lading and bookings | Maersk Container Tracking | 20 | latest event, vessel and voyage, next planned event, route, arrival date (ETA), every container on a B/L |
| USPS, China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail, La Poste, Correos, PostNL, Australia Post, Canada Post, Japan Post, J&T, Aramex and 1,000+ more | 17TRACK Package Tracking Scraper | 10 | status, lifecycle category, carriers on the route, origin and destination country, ETA, checkpoints, in 9 languages |

Not supported: orders bought on Amazon itself (Amazon shows those only in Your Orders after sign-in; the Amazon tracker covers Amazon Shipping and Multi-Channel Fulfillment parcels sent by other stores), the full UPS scan list (UPS publishes only the latest update), SPX Thailand and Taiwan, ocean carriers other than Maersk, proof-of-delivery images and signatures, label creation, rates, pickups and claims.

## How to set it up

1. Install the plugin from the Claude directory (claude.ai > Customize > Plugins) or in Claude Code with `claude plugin install trackbuddy-parcel-tracker@imzlab` after adding the marketplace `afzalbuilds/trackbuddy-parcel-tracker`.
2. Open the plugin's **Connectors** tab and connect the **package-tracking** server. It is Apify's official MCP server (`mcp.apify.com`) limited to the nine tracking Actors, so a browser window opens for you to sign in or create a free Apify account. Free accounts include monthly usage credit, so you can try the tracker without adding a card. Claude may note that the server is not a separately registered directory connector; that is expected, because it is Apify's hosted server rather than one Imzlab runs.
3. Ask Claude to track a number.

Upgrading from 1.0: version 1.1 adds four trackers to the connector. If Claude says a tracker is missing, disconnect and reconnect **package-tracking** in the Connectors tab.

In Claude Code, `/mcp` shows the connection state and `/trackbuddy-parcel-tracker:track <numbers>` runs a lookup directly.

## Other AI agents: Codex, Gemini CLI, Copilot and more

The skills and the **package-tracking** server are the same in every app; only the manifest each app reads differs. Every app signs you in to Apify in the browser, and lookups are billed to your own Apify account exactly as in Claude.

| App | Install | What loads |
|---|---|---|
| Claude (chat, Cowork, Claude Code) | [How to set it up](#how-to-set-it-up) | skills, commands, tracking-auditor agent, package-tracking server |
| OpenAI Codex (CLI and app) | [OpenAI Codex](#openai-codex) | skills, package-tracking server |
| Gemini CLI (and Qwen Code) | [Gemini CLI](#gemini-cli) | skills, commands, tracking-auditor subagent, package-tracking server |
| GitHub Copilot CLI and VS Code | [GitHub Copilot](#github-copilot-cli-and-vs-code) | skills, commands, tracking-auditor agent, package-tracking server |
| Grok Build | [Grok Build](#grok-build) | skills, commands, tracking-auditor agent, package-tracking server |
| Cursor and Grok Bot | [Grok Bot and Cursor](#grok-bot-and-cursor) | skills, commands, tracking-auditor agent, package-tracking server |
| Hermes Agent | [Hermes Agent](#hermes-agent) | skills, package-tracking server |
| OpenCode | [OpenCode](#opencode) | skills, commands, tracking-auditor subagent, package-tracking server |
| Antigravity, Kiro, Devin, Factory Droid, Augment, Junie, Mistral Vibe | [More agents](#more-agents) | skills and package-tracking server, plus commands and agent where the app supports them |
| Any other agent with skills support | [More agents](#more-agents) | skills, plus the server URL added by hand |

Checked on 2026-11-16: the plugin installs, and its skills and server load, in Codex 0.167, Gemini CLI 0.68.0, GitHub Copilot CLI 1.0.112 and OpenCode 1.18.27, and it passes the plugin validators of Claude Code, Gemini CLI, Grok Build and Hermes Agent. Apify sign-in and a live lookup were not repeated in those apps. The apps under [More agents](#more-agents) follow each app's published plugin format but were not installed.

### OpenAI Codex

Needs Codex CLI 0.142 or newer (`codex update`). The Codex app uses the same settings, so the plugin appears there too.

```bash
codex plugin marketplace add afzalbuilds/trackbuddy-parcel-tracker
```

```bash
codex plugin add trackbuddy-parcel-tracker@imzlab
```

```bash
codex mcp login package-tracking
```

The last command opens Apify's sign-in page in your browser. Ask Codex to track a number. You can also call a skill by name, for example `$track-package`. Codex loads the skills and the package-tracking server, but not the plugin's commands or the tracking-auditor agent, so ask for a bulk report in plain words there.

### Gemini CLI

```bash
gemini extensions install https://github.com/afzalbuilds/trackbuddy-parcel-tracker
```

When Gemini CLI first connects to the package-tracking server it opens Apify's sign-in page in your browser; if it does not, run `/mcp auth package-tracking` inside Gemini CLI. The extension adds the skills, the `/track` and `/track-report` commands and the tracking-auditor subagent. Update it later with `gemini extensions update trackbuddy-parcel-tracker`.

Qwen Code installs the same extension: `qwen extensions install afzalbuilds/trackbuddy-parcel-tracker`.

### GitHub Copilot CLI and VS Code

```bash
copilot plugin marketplace add afzalbuilds/trackbuddy-parcel-tracker
```

```bash
copilot plugin install trackbuddy-parcel-tracker@imzlab
```

In a Copilot CLI session, run `/mcp auth package-tracking` to sign in to Apify. The plugin adds the skills, the `/track` and `/track-report` commands, the tracking-auditor agent and the package-tracking server.

In VS Code, turn on the `chat.plugins.enabled` setting, run **Chat: Install Plugin from Source** from the Command Palette and enter `afzalbuilds/trackbuddy-parcel-tracker`.

### Grok Build

```bash
grok plugin install afzalbuilds/trackbuddy-parcel-tracker
```

Grok asks you to confirm that you trust the plugin; its tracking server stays off until you do. Then open `/mcps`, select **package-tracking** and press `i` to sign in to Apify. Ask Grok to track a number.

### Grok Bot and Cursor

Marketplace listing is pending. Once listed, add the plugin from the marketplace and sign in to Apify when the tracking server asks.

On grok.com you can use the trackers today without the skills: open Connectors, add a custom connector and paste the tracking server URL below.

### Hermes Agent

Needs Hermes 0.21.6 or newer, which loads Agent Plugins packages (a root `plugin.json`).

```bash
hermes plugins install afzalbuilds/trackbuddy-parcel-tracker --no-enable
```

```bash
hermes plugins enable trackbuddy-parcel-tracker
```

The package declares the tracking server, but Hermes signs in to a remote server only when your own configuration asks for it. Register the server under the same name with browser sign-in, so Hermes uses your Apify account for it. Run this in a terminal, not from a script, because Hermes starts the sign-in only when it can talk to you:

```bash
hermes mcp add package-tracking --auth oauth --url "https://mcp.apify.com/?tools=muhammadafzal/17track-package-tracking-scraper,muhammadafzal/dhl-package-tracking-scraper,muhammadafzal/fedex-package-tracking-scraper,muhammadafzal/ups-package-tracking-scraper,muhammadafzal/multi-carrier-package-tracking,muhammadafzal/amazon-shipping-tracking-scraper,muhammadafzal/gofo-package-tracking-scraper,muhammadafzal/spx-express-package-tracking-scraper,muhammadafzal/maersk-container-tracking-scraper"
```

Hermes opens Apify's consent page in your browser. After you allow access it lists the 13 tools (the nine trackers plus the run, dataset and record tools); press Enter to enable them all. `hermes mcp test package-tracking` confirms the connection, and `hermes mcp login package-tracking` signs in again later if the access is ever revoked.

Every lookup is a paid Actor run on your Apify account, so the package marks the server `untrusted`: Hermes asks before each tracking call and refuses them in unattended runs. Your own server entry replaces the package's entry, including that setting; add `trust: untrusted` under `mcp_servers.package-tracking` in `~/.hermes/config.yaml` to keep the prompts.

To install only one skill instead of the whole package: `hermes skills install afzalbuilds/trackbuddy-parcel-tracker/skills/track-package`. The tracking server still has to be added as above.

### OpenCode

OpenCode has no plugin bundle, so the skills, the server and the extras are added separately:

```bash
npx skills add afzalbuilds/trackbuddy-parcel-tracker -a opencode -g
```

```bash
opencode mcp add package-tracking --url "https://mcp.apify.com/?tools=muhammadafzal/17track-package-tracking-scraper,muhammadafzal/dhl-package-tracking-scraper,muhammadafzal/fedex-package-tracking-scraper,muhammadafzal/ups-package-tracking-scraper,muhammadafzal/multi-carrier-package-tracking,muhammadafzal/amazon-shipping-tracking-scraper,muhammadafzal/gofo-package-tracking-scraper,muhammadafzal/spx-express-package-tracking-scraper,muhammadafzal/maersk-container-tracking-scraper"
```

```bash
opencode mcp auth package-tracking
```

For the `/track` and `/track-report` commands and the tracking-auditor subagent, copy the `opencode/commands` and `opencode/agents` folders of this repository into `~/.config/opencode/`.

### More agents

| App | How to install |
|---|---|
| Google Antigravity | `agy plugin install https://github.com/afzalbuilds/trackbuddy-parcel-tracker` |
| Kiro | Powers panel, **Add Custom Power**, **Import power from GitHub**, then `https://github.com/afzalbuilds/trackbuddy-parcel-tracker` |
| Devin (formerly Windsurf) | `devin plugins install afzalbuilds/trackbuddy-parcel-tracker` |
| Factory Droid | `droid plugin marketplace add afzalbuilds/trackbuddy-parcel-tracker`, then `droid plugin install trackbuddy-parcel-tracker@imzlab --scope user` |
| Augment (Auggie CLI) | `auggie plugin marketplace add afzalbuilds/trackbuddy-parcel-tracker`, then enable the plugin in `/plugins` |
| JetBrains Junie CLI | `/extensions marketplace add afzalbuilds/trackbuddy-parcel-tracker`, then `/extensions install trackbuddy-parcel-tracker` |
| Mistral Vibe | clone this repository into `~/.vibe/plugins/trackbuddy-parcel-tracker`, run `/reload-plugins`, then `/mcp login package-tracking` |
| Cline, Kilo Code, Amp, Goose and other agents that read Agent Skills | `npx skills add afzalbuilds/trackbuddy-parcel-tracker` installs the skills; then add the server URL below as a remote MCP server with OAuth in that app |

### Tracking server URL

Any MCP client can use the trackers directly by adding this URL as a remote (streamable HTTP) server with OAuth:

```
https://mcp.apify.com/?tools=muhammadafzal/17track-package-tracking-scraper,muhammadafzal/dhl-package-tracking-scraper,muhammadafzal/fedex-package-tracking-scraper,muhammadafzal/ups-package-tracking-scraper,muhammadafzal/multi-carrier-package-tracking,muhammadafzal/amazon-shipping-tracking-scraper,muhammadafzal/gofo-package-tracking-scraper,muhammadafzal/spx-express-package-tracking-scraper,muhammadafzal/maersk-container-tracking-scraper
```

## n8n workflow templates

Two ready-made n8n workflows use the same tracking Actors: a form that tracks one package, and a daily Google Sheets check that emails delivery exceptions. Import the JSON files from the [n8n folder](n8n/README.md); they need the Apify node for n8n and your own Apify account.

## Pricing

The plugin itself is free. Each tracked record is billed by Apify on your account at the Actor's pay-per-event price. Prices below are the Apify Store prices on the FREE tier as of 2026-11-13. Paid Apify plans get a discount, and blocked or not-found numbers are never charged as records.

| Tracker | Per tracked record | Per run start | 10 records cost |
|---|---|---|---|
| DHL | USD 0.020 | USD 0.00005 | USD 0.20 |
| FedEx | USD 0.020 | USD 0.005 | USD 0.205 |
| UPS | USD 0.030 | USD 0.005 | USD 0.305 |
| 17TRACK (USPS, China Post, YunExpress and others) | USD 0.039 | USD 0.005 | USD 0.395 |
| Multi-carrier (FedEx International Connect, USPS fallback) | USD 0.040 | USD 0.005 | USD 0.405 |
| Amazon Shipping | USD 0.015 | USD 0.005 | USD 0.155 |
| GOFO | USD 0.010 | USD 0.005 | USD 0.105 |
| SPX Express | USD 0.005 | USD 0.005 | USD 0.055 |
| Maersk containers | USD 0.008 | USD 0.005 | USD 0.085 |

Before any batch over 10 numbers, Claude tells you the estimated cost and asks before running. Ask "what will it cost to track these" to get an estimate without running anything.

## Data and privacy

- Tracking numbers you give Claude are sent to Apify's hosted MCP server (`mcp.apify.com`), which runs the tracking Actor on your Apify account. The Actor queries the carrier's public tracking page.
- Results are stored in your Apify account's datasets for your plan's retention period, where you can download them as CSV, Excel or JSON. The plugin stores nothing itself and sends nothing to any other service.
- Tracking records can include delivery addresses and recipient names shown on the carrier's public page. Treat them as personal data.
- You authorize Apify through OAuth in the plugin's Connectors tab. The plugin itself holds no Apify account details.
- Full policy: https://imzlab.com/privacy#package-tracker. Terms: https://imzlab.com/terms#package-tracker. Support: https://imzlab.com/support#package-tracker

## Frequently asked questions

**Do I need a FedEx, UPS or DHL developer account?** No. The trackers read the same public tracking pages you would open in a browser, and the plugin never asks for carrier API keys.

**How many numbers can I track at once?** Per run: 10 for DHL and 17TRACK, 20 for Maersk, 25 for UPS and the multi-carrier tracker, 30 for FedEx, 200 for Amazon Shipping, 500 for SPX Express and 1,000 for GOFO. Claude splits longer lists into several runs automatically and merges the results.

**What happens with a number that is not found?** It costs nothing as a record and appears in the results table with the carrier's message, for example "no tracking information" or "label created, not yet scanned". New labels can take 24 to 48 hours to show scans.

**Can I track USPS?** Yes, through 17TRACK, which aggregates USPS. If 17TRACK cannot read a number, Claude can retry it on the multi-carrier tracker, which reads USPS's own page on a best-effort basis.

**Can I track an Amazon package?** Yes, when it was shipped with Amazon Shipping or Amazon Multi-Channel Fulfillment by another store (TBA numbers in the US, UK numbers in the United Kingdom). Amazon keeps scan history for 45 days. Orders you placed on Amazon itself are only visible in Your Orders, so those come back as not found, at no charge.

**Can I track a Shopee order?** Yes, if it ships with SPX Express in the Philippines, Malaysia, Indonesia, Vietnam, Singapore or Brazil. Use the SPX tracking number with its country prefix (for example `SPXID…` or `PH…`), not the Shopee order ID.

**Can I track a shipping container?** Yes, for cargo carried by Maersk: give the container number (4 letters and 7 digits), the bill of lading number or the booking number. You get the latest event, the vessel, the next planned move and the estimated arrival date. Other ocean lines are not covered.

**Can it track AliExpress, Temu or Shein orders?** Yes. Those parcels travel on carriers such as YunExpress, Cainiao, 4PX, Yanwen and China Post, all covered by 17TRACK. Use the tracking number from the order page, not the order number.

**Is this an official carrier tool?** No. This plugin and the Actors are independent and not affiliated with DHL, FedEx, UPS, USPS, Amazon, GOFO, SPX Express, Shopee, Maersk or 17TRACK. Status wording comes from the carriers.

**Can I schedule a daily check?** In Claude Code or Cowork, ask the tracking auditor agent to run your list and report exceptions. For unattended schedules, create an Apify schedule for the Actor in Apify Console.

## Skills and commands in this plugin

- **track-package**: single or mixed-carrier lookups with automatic carrier detection.
- **track-container**: Maersk containers, bills of lading and bookings, with vessel, next planned event and arrival date.
- **bulk-tracking-report**: paste a list or a CSV column, get a status table, exceptions first, plus a downloadable CSV.
- **delivery-exceptions**: triage delayed, failed, returned and held shipments and draft customer replies.
- **tracking-cost-estimate**: estimate the Apify cost of a batch before running it.
- **/trackbuddy-parcel-tracker:track** and **/trackbuddy-parcel-tracker:track-report** commands (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode).
- **tracking-auditor** agent (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode): runs a list and writes an exceptions report.

Codex and Hermes Agent load the skills and the package-tracking server only; there, ask for a bulk report in plain words. In Codex the skills can also be called by name, for example `$track-package`.

## Source and support

Source: https://github.com/afzalbuilds/trackbuddy-parcel-tracker. Issues and feature requests: https://github.com/afzalbuilds/trackbuddy-parcel-tracker/issues. The Actors are published on Apify Store by Imzlab Technologies under the `muhammadafzal` account.

Last verified: 2026-11-16. Live lookups tested in Claude and Hermes Agent 0.23.1; install and loading checked in Codex 0.167, Gemini CLI 0.68.0, GitHub Copilot CLI 1.0.112 and OpenCode 1.18.27; Grok Build's plugin validator passes.
