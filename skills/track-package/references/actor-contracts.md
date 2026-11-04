# Actor contracts

Verified against live runs on 2026-11-03. Prices are Apify Store pay-per-event prices on the FREE tier; paid Apify plans are 2.5 to 20 percent lower. Only records written to the dataset are charged; every run also charges one start event per started GB of memory.

All Actors belong to Apify user `muhammadafzal`. On the connector, the tool name is the Actor ID with `/` replaced by `--`, for example `muhammadafzal--dhl-package-tracking-scraper`. Each tool returns the run (`runId`, `status`, `storages.datasets.default.id`, `storages.keyValueStores.default.id`), not the rows. If a tool response already includes dataset items, use them; otherwise fetch them with `get-dataset-items`.

Run options (`memory`, `timeout`, `maxTotalChargeUsd`) go in the tool's `callOptions`, never inside `input`.

## Shared rules

- Not-found, blocked, invalid and failed numbers never produce a dataset row. They are counted and explained in the key-value record named below.
- Lookups run one at a time inside a run; allow 10 to 60 seconds per number.
- Do not pass proxy settings; every Actor manages its own access.

---

## DHL: `dhl-package-tracking-scraper`

Price: USD 0.020 per record (GOLD 0.017), start USD 0.00005. Max 10 numbers per run. Typical duration: 10 to 30 s for the first number, a few seconds for each next one.

Input:
```json
{
  "trackingNumbers": ["1234567890"],
  "demoMode": false,
  "accessMode": "auto",
  "language": "en",
  "requesterCountryCode": "US",
  "includeRaw": false
}
```
- Always send `demoMode: false` for real numbers. DHL's sample number `7777777770` with `demoMode: true` returns a labelled demo record; mixing it with real numbers rejects the run.
- `accessMode`: `auto` uses DHL's public site, or the official API if the user supplied `dhlApiKey` (a developer.dhl.com Consumer Key, passed as the `dhlApiKey` input). `api` requires the key.
- `recipientPostalCode` (string) only with exactly one number, when DHL asks for destination verification.
- `language` is a 2-letter ISO code, `requesterCountryCode` a 2-letter country code.

Dataset row (camelCase): `trackingNumber`, `shipmentId`, `service` (`express`, `ecommerce`, `parcel-de`, `dsc`…), `productName`, `statusCode` (`pre-transit` | `transit` | `delivered` | `failure` | `unknown`), `status` (DHL text such as `DELIVERED`, `OUT FOR DELIVERY`), `description`, `statusTimestamp`, `estimatedDelivery`, `originCountryCode`, `originPostalCode`, `originCity`, `destinationCountryCode`, `destinationPostalCode`, `destinationCity`, `currentLocation`, `totalPieces`, `events[]` of `{timestamp, statusCode, status, description, location}` (newest first), `serviceUrl`, `sourceUrl`, `retrievedAt`, `isDemoData`, `raw`.

Key-value record `SUMMARY`: `outcome` (`data` | `demo` | `empty` | `blocked` | `failed` | `rejected`), `requested`, `delivered`, `demoRecords`, `empty`, `blocked`, `failed`, `notStarted`, `chargeLimitReached`, `accessRoute`, `warnings` (strings formatted `"<number>: <message>"`), `finishedAt`. Run status is SUCCEEDED even for empty or rejected runs.

---

## FedEx: `fedex-package-tracking-scraper`

Price: USD 0.020 per record (GOLD 0.017), start USD 0.005 per started GB (default memory 1 GB, so one event). Max 30 numbers per run. About 6 to 10 s per number; pass `callOptions.timeout: 900` for more than 15 numbers.

Input:
```json
{
  "trackingNumbers": ["794644790138", "449044304137"],
  "includeDetailedScans": true,
  "maxResults": 30
}
```
- `trackingNumbersText` (string) accepts a pasted list separated by newlines, commas, semicolons or spaces; it is merged with `trackingNumbers`.
- `includeDetailedScans: false` returns only the latest scan.

Dataset row (snake_case): `tracking_number`, `carrier` ("FedEx"), `status`, `status_code` (`DL` delivered, `IT` in transit, `OD` out for delivery, `DE` exception, `PU` picked up, `OC`/`IN` label created, `HL` held at location, `RS` return to shipper, `DY` delay, `CA` cancelled), `status_detail`, `estimated_delivery_date`, `actual_delivery_date`, `service`, `ship_date`, `origin` and `destination` (`{city, state_or_province, postal_code, country}` or null), `package_count`, `last_scan`, `scan_events[]` of `{timestamp, status, location, description, event_code}` (newest first), `special_handling[]`, `tracking_url`, `source`, `scraped_at`, `warnings[]`.

Key-value record `SUMMARY`: `outcome` (`DATA` | `DATA_WITH_WARNINGS` | `EMPTY` | `BLOCKED` | `REJECTED`), `requested`, `records`, `warnings` (strings `"<number>: <message>"`), `blocked_requests`, `charged_result_events`. Run status is SUCCEEDED even with zero records.

Data-quality rule: if `status` is the literal word `Status`, or `ship_date` is `Ship date`, or `actual_delivery_date` is `Delivered date`, FedEx returned its page shell without data (usually an expired or unknown number). Treat the row as `unknown` with the note "FedEx returned no details; the number may be older than FedEx's public history" and do not display those placeholder strings.

---

## UPS: `ups-package-tracking-scraper`

Price: USD 0.030 per record (GOLD 0.022), start USD 0.005 per started GB (default 512 MB, one event). Max 25 numbers per run. 10 to 60 s per number, plus retries; pass `callOptions.timeout: 900` for more than 10 numbers.

Input:
```json
{
  "trackingNumbers": ["1Z999AA10123456784"],
  "locale": "en_US",
  "requestTimeoutSecs": 45
}
```
- `locale` is `language_COUNTRY`, for example `en_GB`, and also selects the proxy country.

Dataset row (camelCase): `trackingNumber`, `carrier` ("UPS"), `status` (`pre_advice` | `accepted` | `in_transit` | `out_for_delivery` | `delayed` | `delivery_attempted` | `ready_for_pickup` | `delivered` | `returned` | `exception` | `unknown`), `statusText`, `terminal`, `delivered`, `delayed`, `estimatedDelivery` (free text such as "Friday, October 23 by 7:00 P.M."), `lastEvent` (`{text, location}` or null), `deliveredTo`, `receivedBy`, `sourceUrl`, `scrapedAt`. No event history.

Key-value record `OUTPUT`: `outcome` (`DATA` | `EMPTY` | `BLOCKED` | `REJECTED` | `FAILED`), `attempted`, `delivered`, `notFound`, `invalidInput`, `blocked`, `failed`, `duplicatesRemoved`, `warnings[]` of `{trackingNumber (masked), category (not_found | invalid_input | blocked | failed | charge_limit), message}`, `startedAt`, `finishedAt`.

Data-quality rule: when `statusText` is `No Information Available` but `delivered` is true and `deliveredTo` or `receivedBy` is filled, report "Delivered" with the delivered-to location and note that UPS no longer shows the scan detail. When `statusText` is `No Information Available` and `delivered` is false, report `unknown`.

---

## 17TRACK: `17track-package-tracking-scraper`

Price: USD 0.039 per record (GOLD 0.0312), start USD 0.005. Max 10 numbers per run. Hard 120 s work budget per run; a batch of 10 may stop early with `stopReason: runtime_limit` and the rest `skipped` (rerun them).

Input:
```json
{
  "trackingNumbers": ["RB123456789CN"],
  "language": "en",
  "carrier": 0,
  "requestTimeoutSecs": 45,
  "includeRaw": false
}
```
- `carrier`: 0 for auto-detect, otherwise one 17TRACK carrier code for the whole run (see carrier-formats.md).
- `language`: `en`, `de`, `es`, `fr`, `it`, `nl`, `pl`, `pt`, `tr`.

Dataset row (camelCase): `trackingNumber`, `displayTrackingNumber`, `carrierIds[]`, `carriers[]`, `status` (text), `subStatus`, `statusDescription`, `statusCategory` (`not_found` | `info_received` | `in_transit` | `pickup` | `out_for_delivery` | `undelivered` | `delivered` | `alert` | `expired` | `unknown`), `daysInTransit`, `originCountry`, `destinationCountry`, `estimatedDelivery`, `latestEvent` (`{timestamp, description, location, carrier}`), `events[]` (same shape), `sourceUrl`, `retrievedAt`, `isDemoData`, `raw`.

Key-value record `OUTPUT`: `outcome` (`DATA` | `DEMO` | `EMPTY` | `BLOCKED` | `FAILED` | `REJECTED` | `LIMIT_REACHED`), `requested`, `attempted`, `delivered`, `empty`, `blocked`, `failed`, `skipped`, `invalidInput`, `stopReason` (`completed` | `invalid_input` | `item_limit` | `charge_limit` | `runtime_limit` | `source_failure` | `free_plan_limit` | `execution_error`), `warnings[]` of `{trackingNumber (masked), category (invalid_input | empty | blocked | failed), message}`. The run ends FAILED when the outcome is BLOCKED or FAILED; still read OUTPUT.

Behaviour: the first blocked or failed number stops the batch and the remaining numbers are `skipped` (not charged). Rerun skipped numbers in a new run. Public demo number `7777777770` returns a labelled demo record.

Data-quality rule: `latestEvent` and `statusDescription` are reliable. The `events[]` list can arrive with the timestamp and description concatenated into one string (for example a `timestamp` of `2022-03-25 19:57CHINA,WUHAN, Processed…`). Only display `events[]` when every `timestamp` parses as a date on its own; otherwise show `latestEvent` and say the full history is available on the 17TRACK page at `sourceUrl`.

---

## Multi-carrier: `multi-carrier-package-tracking`

Price: USD 0.040 per record from 2026-11-06 (GOLD 0.032; USD 0.020 before that date), start USD 0.005. Max 25 numbers per run; free Apify plans are capped at 5 results per run. All lookups stop after about 4 minutes. Supported: UPS (`1Z…`) and FedEx International Connect (`KOT…`/`PRO…`). Not supported: USPS (always blocked) and standard FedEx (12 or 15 digits, fails).

Input:
```json
{
  "trackingNumbers": ["1Z999AA10123456784"],
  "carrier": "UPS",
  "includeDetailedEvents": true,
  "requestTimeoutSecs": 25
}
```
- `carrier`: `auto` | `USPS` | `UPS` | `FedEx`. Pass the carrier explicitly; `auto` rejects unrecognized formats as `ambiguous_carrier`.

Dataset row (camelCase): `trackingNumber`, `carrier`, `status` (same enum as the UPS Actor), `statusText`, `terminal`, `delivered`, `delayed`, `estimatedDelivery`, `actualDelivery` (free text such as `12/05/2025 10:29 A.M.`), `service`, `origin` and `destination` (`{city, stateOrProvince, postalCode, country}` or null), `lastEvent` (`{timestamp, status, description, location}`), `events[]` (same shape, newest first), `sourceUrl`, `scrapedAt`.

Key-value record `OUTPUT`: `outcome` (`DATA` | `EMPTY` | `BLOCKED` | `REJECTED` | `FAILED`), counts as UPS plus `requested`, `warnings[]` of `{trackingNumber (masked), carrier, category (not_found | invalid_input | ambiguous_carrier | blocked | failed | charge_limit), message}`, `stopReason` (`completed` | `charge_limit` | `runtime_limit` | `runtime_guard` | `provider_unavailable` | `platform_shutdown` | `runtime_error` | `invalid_input`), `provider`, `effectiveLimit`. The run ends FAILED when the outcome is FAILED, BLOCKED or REJECTED; still read OUTPUT.

Event timestamps are UPS free text (`12/05/2025 10:29 A.M.`), not ISO; a `status` like "A late flight has caused a delay…" inside `events[]` marks a delay.

---

## Cost formula

cost = runs × start fee + delivered records × per-record price.

Examples on the FREE tier: 1 UPS number ≈ USD 0.035; 10 DHL numbers ≈ USD 0.20; 30 FedEx numbers ≈ USD 0.605; 25 UPS numbers ≈ USD 0.755; 10 17TRACK numbers ≈ USD 0.395; 100 mixed numbers (40 FedEx, 30 UPS, 20 DHL, 10 USPS via 17TRACK) ≈ USD 0.805 + 0.905 + 0.40 + 0.395 ≈ USD 2.51.

Quote costs as "about", because not-found numbers are not charged and the user's Apify plan tier may be discounted.
