---
name: track-package
description: Track a package, parcel, shipment or order by tracking number and report its delivery status. Use when the user says "track package", "track my package", "where is my package", "where is my order", "track my order", "track shipment", "tracking number", "delivery status", "has it been delivered", or pastes one or more tracking numbers from DHL, FedEx, UPS, USPS, Amazon Shipping (TBA numbers), GOFO, SPX Express (Shopee Express), China Post, YunExpress, Cainiao, 4PX, Yanwen, Royal Mail or any other carrier, including AliExpress, Temu, Shein, Shopee, eBay and Shopify orders. Also use when the user asks for the scan history, checkpoints or estimated delivery of a parcel.
---

# Track a package

You track parcels by running Imzlab Technologies' tracking Actors on Apify through the **package-tracking** MCP server (some apps call it a connector). Each Actor is a tool on that server whose name contains the Actor slug, for example `muhammadafzal--fedex-package-tracking-scraper`; depending on the app, the tool name may carry a server prefix or use underscores in place of hyphens, so match on the slug. The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`, which you need to collect results.

Read `references/actor-contracts.md` for the exact input and output of every Actor before your first call in a conversation. `references/carrier-formats.md` has the routing rules and `references/status-vocabulary.md` the normalized status mapping.

Ocean containers and bills of lading (Maersk) have their own skill, `track-container`.

## Steps

1. **Collect the numbers.** Extract every tracking number from the message, file or pasted text. Strip spaces and hyphens inside a number, uppercase it, and drop duplicates. Ignore order numbers (Shopify `#1234`, Amazon `112-…`, AliExpress 16-digit order IDs, Shopee order IDs) and tell the user they are not tracking numbers. If nothing looks like a tracking number, ask for one.

2. **Detect the carrier for each number** with the rules in `references/carrier-formats.md`. If the user named the carrier, trust the user over the format. If a number is ambiguous (for example 12 digits could be FedEx or a UPS InfoNotice), use the user's wording, then the default in the table, and say which you chose.

3. **Group by Actor and respect per-run limits**: DHL 10, 17TRACK 10, UPS 25, multi-carrier 25 (at most 3 USPS numbers), FedEx 30, Amazon Shipping 200, SPX Express 500, GOFO 1,000. Split larger groups into several runs.

4. **Estimate the cost** from the price table in `references/actor-contracts.md`. For 1 to 10 numbers, run immediately and report the cost afterwards. For more than 10 numbers, or an estimate above USD 0.50, show the estimate first and run only after the user confirms. For a long list, prefer the `bulk-tracking-report` skill.

5. **Run the Actors.** Call each Actor tool with the input shape from the contract file. Runs for different Actors can start in parallel. Always send `demoMode: false` to DHL unless the user explicitly asks for the DHL demo. Set a run `timeout` of 900 seconds for FedEx runs over 15 numbers and 300 seconds for GOFO runs over 500 numbers.

6. **Wait for every run to finish.** A tool response with status `RUNNING` or `READY` is not a result. Call `get-actor-run` with the `runId` and `waitSecs: 45`, repeat until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`, up to 8 times. Never report a run as done while it is still running, and never invent a status.

7. **Fetch the results**, whatever the run status was:
   - `get-dataset-items` with the run's default `datasetId` (use `limit` equal to the number of requested numbers, at least 20).
   - `get-key-value-store-record` with the run's default `keyValueStoreId` and the key named in the contract file: `SUMMARY` for DHL and FedEx, `OUTPUT` for every other Actor.
   A run can end `SUCCEEDED` with zero items, and several Actors end `FAILED` when every number was blocked yet still carry useful explanations. The record's `outcome` and its warnings or number lists explain every number that is missing from the dataset.

8. **Reconcile.** Every requested number must appear in your answer exactly once. Match dataset rows by their tracking number field. Any requested number without a row gets its status from the summary record: `not_found` (empty, no tracking information), `blocked` (carrier denied access, try again later), `invalid` (rejected input) or `failed` (lookup error). Warnings in 17TRACK, UPS and multi-carrier mask the number, for example `1Z**********6784`; match on the visible prefix and suffix. GOFO, SPX and Amazon list the numbers unmasked. Amazon writes a row for every number, so read `lookupStatus` on each row instead. A GOFO row can answer two requested numbers (waybill and `CR…` reference of one parcel).

9. **Normalize** each row to the shared schema in `references/status-vocabulary.md` and apply the data-quality rules in `references/actor-contracts.md` (placeholder values, garbled event lists, misleading status text, Amazon's oldest-first events).

10. **Answer** with one table and a short cost line, as described below. If any number came back blocked, offer to retry it once; if a number was not found and the label is less than 48 hours old, say scans usually appear within 24 to 48 hours.

## Answer format

Give one table with these columns: Tracking number, Carrier, Status, Last update (time and location), ETA or delivered time, Notes. Put exceptions, delays and not-found numbers at the top. Then one line: "Apify cost: about USD X for N records in M runs, billed to your Apify account." Then, only if the user asked for history or there is a single package, list the events newest first, each as time, location, description.

Keep the carrier's own wording for the status text and add the normalized status in parentheses when they differ, for example "Out For Delivery Today (out for delivery)". Give times exactly as the carrier reports them and say when they are UTC (Amazon, SPX, GOFO's `…Utc` fields); do not convert time zones unless the user asks. Do not paste raw JSON. Do not show `receivedBy` names, Amazon's `shipperName` or SPX's `clientOrderId` unless the user asks for them.

## When things go wrong

- The package-tracking server is not connected or the tool call returns an authorization error: tell the user to sign in to Apify (a free account works) for the package-tracking server in their app's connector or MCP settings, then retry. In Claude this is the plugin's Connectors tab.
- An Actor tool for a carrier is missing from the server: the connection was made before that carrier was added. Ask the user to disconnect and reconnect the package-tracking server, and meanwhile try the number on 17TRACK.
- The run reports `charge_limit` or `chargeLimitReached`: the user's Apify max charge per run was hit; report which numbers were skipped and suggest raising the limit in Apify Console or splitting the batch.
- The run reports `provider_unavailable`, `BLOCKED` for every number, or `source_failure`: the carrier blocked the lookup; report it plainly, charge is zero for those numbers, and offer to retry once or use a different tracker (17TRACK can often track a UPS, FedEx, DHL, GOFO or SPX number too).
- Multi-carrier must never be used for standard FedEx numbers (12 or 15 digits); route those to the FedEx Actor. Use it for FedEx International Connect, and for USPS only as the fallback described in `references/carrier-formats.md`.
- An Amazon number comes back `NOT_FOUND`: if the user bought the item on Amazon itself, explain that Amazon only shows those orders in Your Orders after sign-in; the public tracker covers Amazon Shipping and Multi-Channel Fulfillment parcels sent by other stores.
- No Actor returns the full UPS scan list. Report the latest update and give the UPS page link from `sourceUrl`.
