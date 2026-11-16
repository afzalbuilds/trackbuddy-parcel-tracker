---
description: Track one or more packages by tracking number (DHL, FedEx, UPS, USPS, Amazon Shipping, GOFO, SPX, 17TRACK carriers) or Maersk containers
argument-hint: <tracking numbers, separated by spaces or commas>
---

Track these tracking numbers using the `track-package` skill from this plugin (and the `track-container` skill for Maersk container, bill of lading or booking numbers): $ARGUMENTS

Detect each carrier, run the matching Apify tracking Actor through the package-tracking MCP server, wait for every run to finish, read the dataset and the SUMMARY or OUTPUT record, and present one table with status, last update, ETA and notes, followed by the Apify cost line. If no tracking numbers were given, ask for them.
