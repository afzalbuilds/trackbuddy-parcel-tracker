---
description: Track a whole list or CSV of tracking numbers and write a status report with exceptions first
argument-hint: <path to CSV/XLSX, or pasted list of tracking numbers>
---

Build a bulk tracking report using the `bulk-tracking-report` skill from this plugin for: $ARGUMENTS

If the argument is a file path, read it and find the tracking-number column; otherwise treat the argument as a pasted list. Show the run plan and estimated Apify cost, wait for confirmation when there are more than 10 numbers, run the Actors, reconcile every number, and write the exceptions-first report plus a CSV next to the input file (or as a downloadable file in chat).
