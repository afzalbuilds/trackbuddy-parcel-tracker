---
name: bulk-tracking-report
description: Track many packages at once and produce a status report. Use when the user pastes a list of tracking numbers, attaches a CSV, spreadsheet or order export (Shopify, WooCommerce, Amazon, eBay, Etsy, ShipStation), or asks to "track multiple packages", "bulk track", "track all these", "check which orders are delivered", "which shipments are late", "daily shipment audit" or "export tracking statuses to a spreadsheet".
---

# Bulk tracking report

Builds on the `track-package` skill: read its `references/actor-contracts.md`, `references/carrier-formats.md` and `references/status-vocabulary.md` first. This skill adds list handling, batching across runs, and a report format.

## Steps

1. **Extract the list.** From pasted text, a CSV or spreadsheet, take the column that holds tracking numbers (look for headers such as tracking, tracking_number, tracking number, shipment, carrier tracking code, AWB, waybill, container, B/L). Keep any adjacent columns the user will want back in the report: order number, customer name, email, ship date, carrier column. Normalize the numbers as in `track-package` and remove duplicates while remembering which order each duplicate belonged to.

2. **Route and batch.** Detect each number's carrier; if the sheet has a carrier column, prefer it. Group by Actor and split into runs of at most: DHL 10, 17TRACK 10, Maersk 20, UPS 25, multi-carrier 25, FedEx 30, Amazon Shipping 200, SPX Express 500, GOFO 1,000. Keep the FedEx Actor for standard FedEx numbers and 17TRACK for USPS; multi-carrier is only for FedEx International Connect and as the USPS fallback (at most 3 USPS numbers per run).

3. **Quote before running.** Show a one-line plan: "N numbers: a FedEx (b runs), c UPS (d runs), e DHL (f runs), g Amazon Shipping (h runs), i other via 17TRACK (j runs). Estimated Apify cost about USD X, roughly Y minutes." List only the carriers that are present. Wait for the user's go-ahead when there are more than 10 numbers or the estimate exceeds USD 0.50. For more than 100 numbers, suggest doing it in chunks of 100 so results arrive sooner.

4. **Run.** Start runs for different Actors in parallel. Start up to 3 runs of the same Actor at a time; start the next when one finishes. Set `callOptions.timeout: 900` for FedEx runs over 15 numbers and `300` for GOFO runs over 500. Poll each run with `get-actor-run` (`waitSecs: 45`) until it is terminal. Tell the user you are waiting if it takes longer than a minute; never report partial results as final without saying so.

5. **Collect and reconcile** every run: dataset rows plus the `SUMMARY`/`OUTPUT` record, as in `track-package`. Every input number appears exactly once in the report. Rerun numbers that a 17TRACK run marked `skipped`, that a run left `notStarted`, `not_started`, `notChecked` or `notLookedUp`, or that Maersk lists under `noResultsUnconfirmed`, up to one extra pass, and report any that still have no result.

6. **Normalize** with the status vocabulary and the data-quality rules.

7. **Report.** Produce, in this order:
   - A summary line: counts per normalized status (delivered, in transit, out for delivery, delayed, exception, not found, blocked) and the total Apify cost.
   - An **Action needed** table first: exceptions, delivery attempts, returns, delays past the ETA, and not-found numbers older than 48 hours. Columns: Order (if known), Tracking number, Carrier, Status, Last update, Location, Suggested action.
   - The **full table** of all numbers, same columns plus ETA/delivered time.
   - A CSV with every row (all input columns preserved, plus `status`, `statusText`, `lastEventTime`, `lastEventLocation`, `estimatedDelivery`, `actualDelivery`, `carrier`, `sourceUrl`, `note`). When you can write files (a coding agent or a desktop workspace), write it next to the input file as `<input-name>-tracking-<YYYY-MM-DD>.csv`; in a chat-only app, give it as a downloadable file, or as a code block if that is not possible.

8. Offer follow-ups: draft customer messages for the exceptions (`delivery-exceptions` skill), or rerun only the non-terminal numbers later.

## Rules

- Never fabricate a status. A number with no result shows "not found", "blocked" or "not attempted" with the Actor's message.
- Sort "Action needed" by severity: exception, delivery attempted, returned, delayed, not found, blocked.
- For Maersk containers use the columns from the `track-container` skill (vessel, next planned event, arrival date) in a separate table; do not force ocean shipments into the parcel table.
- A shipment whose ETA has passed and whose status is still in transit is "delayed" even if the carrier does not say so; note the ETA that was missed.
- Keep personal data minimal: include recipient names or addresses only when they were in the user's input or the user asks.
