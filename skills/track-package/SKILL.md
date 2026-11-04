---
name: track-package
description: Track a package, parcel, shipment or order by tracking number and report its delivery status. Use when the user says "track package", "track my package", "where is my package", "where is my order", "track my order", "track shipment", "tracking number", "delivery status", "has it been delivered", or pastes one or more tracking numbers from DHL, FedEx, UPS, USPS, China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail or any other carrier, including AliExpress, Temu, Shein, eBay and Shopify orders. Also use when the user asks for the scan history, checkpoints or estimated delivery of a parcel.
---

# Track a package

You track parcels by running Imzlab Technologies' tracking Actors on Apify through the plugin's **Apify** connector (the official Apify MCP server). Run an Actor with the connector's `call-actor` tool, giving `actor` as the Actor's full name, for example `muhammadafzal/fedex-package-tracking-scraper`, and `input` from the contract file. The connector also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`, which you need to collect results. Do not call `search-actors` or `fetch-actor-details` first: the five Actor names and their input schemas are in the contract file.

Read `references/actor-contracts.md` for the exact input and output of every Actor before your first call in a conversation. `references/carrier-formats.md` has the routing rules and `references/status-vocabulary.md` the normalized status mapping.

## Steps

1. **Collect the numbers.** Extract every tracking number from the message, file or pasted text. Strip spaces and hyphens inside a number, uppercase it, and drop duplicates. Ignore order numbers (Shopify `#1234`, Amazon `112-…`, AliExpress 16-digit order IDs) and tell the user they are not tracking numbers. If nothing looks like a tracking number, ask for one.

2. **Detect the carrier for each number** with the rules in `references/carrier-formats.md`. If the user named the carrier, trust the user over the format. If a number is ambiguous (for example 12 digits could be FedEx or a UPS InfoNotice), use the user's wording, then the default in the table, and say which you chose.

3. **Group by Actor and respect per-run limits**: DHL 10, 17TRACK 10, UPS 25, multi-carrier 25, FedEx 30 numbers per run. Split larger groups into several runs.

4. **Estimate the cost** from the price table in `references/actor-contracts.md`. For 1 to 10 numbers, run immediately and report the cost afterwards. For more than 10 numbers, or an estimate above USD 0.50, show the estimate first and run only after the user confirms. For a long list, prefer the `bulk-tracking-report` skill.

5. **Run the Actors.** Call `call-actor` once per run with `actor` set to the full Actor name, `input` in the shape from the contract file, `waitSecs: 45`, and run options such as `timeout` in `callOptions`. Runs for different Actors can start in parallel. Always set `demoMode: false` for DHL unless the user explicitly asks for the DHL demo. Set `callOptions.timeout` to 900 seconds for FedEx runs over 15 numbers and UPS runs over 10 numbers.

6. **Wait for every run to finish.** A tool response with status `RUNNING` or `READY` is not a result. Call `get-actor-run` with the `runId` and `waitSecs: 45`, repeat until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`, up to 8 times. Never report a run as done while it is still running, and never invent a status.

7. **Fetch the results**, whatever the run status was:
   - `get-dataset-items` with the run's default `datasetId` (use `limit` equal to the number of requested numbers, at least 20).
   - `get-key-value-store-record` with the run's default `keyValueStoreId` and the key named in the contract file: `SUMMARY` for DHL and FedEx, `OUTPUT` for 17TRACK, UPS and multi-carrier.
   A run can end `SUCCEEDED` with zero items, and 17TRACK or multi-carrier can end `FAILED` yet still carry useful warnings. The record's `outcome` and `warnings` explain every number that is missing from the dataset.

8. **Reconcile.** Every requested number must appear in your answer exactly once. Match dataset rows by their tracking number field. Any requested number without a row gets its status from the warnings: `not_found` (empty, no tracking information), `blocked` (carrier denied access, try again later), `invalid` (rejected input) or `failed` (lookup error). Warnings in 17TRACK, UPS and multi-carrier mask the number, for example `1Z**********6784`; match on the visible prefix and suffix.

9. **Normalize** each row to the shared schema in `references/status-vocabulary.md` and apply the data-quality rules in `references/actor-contracts.md` (placeholder values, garbled event lists, misleading status text).

10. **Answer** with one table and a short cost line, as described below. If any number came back blocked, offer to retry it once; if a number was not found and the label is less than 48 hours old, say scans usually appear within 24 to 48 hours.

## Answer format

Give one table with these columns: Tracking number, Carrier, Status, Last update (time and location), ETA or delivered time, Notes. Put exceptions, delays and not-found numbers at the top. Then one line: "Apify cost: about USD X for N records in M runs, billed to your Apify account." Then, only if the user asked for history or there is a single package, list the events newest first, each as time, location, description.

Keep the carrier's own wording for the status text and add the normalized status in parentheses when they differ, for example "Out For Delivery Today (out for delivery)". Give times exactly as the carrier reports them; do not convert time zones unless the user asks. Do not paste raw JSON. Do not show `receivedBy` names unless the user asks who signed.

## When things go wrong

- The Apify connector is not connected or the tool call returns an authorization error: tell the user to open the plugin's Connectors tab and connect Apify (free account), then retry. If they already use the Apify connector from the Claude directory, that connection is the same one.
- The run reports `charge_limit` or `chargeLimitReached`: the user's Apify max charge per run was hit; report which numbers were skipped and suggest raising the limit in Apify Console or splitting the batch.
- The run reports `provider_unavailable`, `BLOCKED` for every number, or `source_failure`: the carrier blocked the lookup; report it plainly, charge is zero for those numbers, and offer to retry once or use a different tracker (17TRACK can often track a UPS, FedEx or DHL number too).
- Multi-carrier must never be used for USPS or standard FedEx numbers (12 or 15 digits); it only supports UPS and FedEx International Connect. Route USPS to 17TRACK and standard FedEx to the FedEx Actor.
