# Package Tracker: DHL, FedEx, UPS, USPS & 17TRACK — privacy content

Imzlab is replacing per-product privacy pages with one generic privacy page for all products. `plugin.json` points at https://imzlab.com/privacy; if the generic page gets a different path, change `privacyPolicyUrl` there and the "Full policy" link in README.md. The section below is the plugin-specific content to add to that generic page (as a product section) once it exists. Anthropic's reviewer opens the URL, so it must be live before the directory submission.

---

## Package Tracker for Claude

Package Tracker: DHL, FedEx, UPS, USPS & 17TRACK ("the plugin", plugin id `trackbuddy-parcel-tracker`) is a Claude plugin published by Imzlab Technologies Private Limited. The plugin is a set of instructions (skills) for Claude plus a reference to Apify's hosted MCP server. It has no server of its own.

### What the plugin does with your data

- **Tracking numbers.** When you ask Claude to track a package, the tracking numbers you provide are sent by Claude to Apify's MCP server at `mcp.apify.com`, which starts a tracking Actor run on your own Apify account. The Actor queries the carrier's public tracking page (DHL, FedEx, UPS, 17TRACK) or, if you supply your own DHL developer key, DHL's official tracking API.
- **Results.** Tracking results are written to datasets and key-value stores in your Apify account and returned to Claude for display. They may contain delivery addresses, recipient names and scan locations as shown on the carrier's public tracking page.
- **Optional inputs.** A DHL API key or recipient postal code you choose to provide is passed to the DHL Actor for that run only and is never written to results or logs by the Actor.

### What we do not do

- We do not operate any server that receives your data. The only network destination is Apify (apify.com), under Apify's own privacy policy (https://apify.com/privacy-policy).
- We do not collect, store or see your conversations, tracking numbers, tracking results, Apify token or account details.
- We do not send data to any service other than Apify, and we do not use advertising or analytics in the plugin.
- We do not access Claude's memory or chat history.

### Retention

Results stay in your Apify account for your Apify plan's data-retention period (unnamed storages are deleted automatically after that period) and you can delete them at any time in Apify Console. The plugin itself retains nothing.

### Authentication

You authorize Apify through OAuth in the plugin's Connectors tab. The plugin never handles your Apify API token.

### Usage statistics

Imzlab Technologies sees aggregate, anonymous usage of its Apify Actors (run counts and billed events) through Apify Console, and aggregate plugin install and run counts through Anthropic's developer portal. Neither includes tracking numbers or results.

### Children

The plugin is not intended for people under 18.

### Changes and contact

We will update this page when data practices change. Questions: info@imzlab.com.
