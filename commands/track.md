---
description: Track one or more packages by tracking number (DHL, FedEx, UPS, USPS, 17TRACK carriers)
argument-hint: <tracking numbers, separated by spaces or commas>
---

Track these tracking numbers using the `track-package` skill from this plugin: $ARGUMENTS

Detect each carrier, run the matching Apify tracking Actor through the apify-parcel-tracking connector, wait for every run to finish, read the dataset and the SUMMARY or OUTPUT record, and present one table with status, last update, ETA and notes, followed by the Apify cost line. If no tracking numbers were given, ask for them.
