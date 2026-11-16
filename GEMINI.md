# Package Tracker: DHL, FedEx, UPS, USPS & 17TRACK

This extension tracks parcels and Maersk containers with Imzlab Technologies' tracking Actors on Apify, through the `package-tracking` MCP server. The full instructions live in the extension's skills. Activate the matching skill before you call any of the server's tools:

- `track-package`: one or more tracking numbers from any carrier.
- `track-container`: Maersk container, bill of lading or booking numbers.
- `bulk-tracking-report`: a list or CSV of numbers, as an exceptions-first report.
- `delivery-exceptions`: late, failed, held or returned parcels and customer replies.
- `tracking-cost-estimate`: the Apify cost of a batch before running it.

Every tool call on `package-tracking` starts a paid Actor run on the user's own Apify account. Follow the skill's cost rules: estimate from the price table in its references, ask before large batches, and never quote a cost from a run's usage figures. Wait for each run to finish, account for every requested item, and never invent a result.

If the tracking tools are missing, the server is not signed in yet: ask the user to run `/mcp auth package-tracking` and approve Apify's sign-in page in the browser.

Commands: `/track-report`, `/track`. Subagent for unattended runs that leave a report on disk: `tracking-auditor`.
