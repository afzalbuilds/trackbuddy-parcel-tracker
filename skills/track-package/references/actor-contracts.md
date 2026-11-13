# Actor contracts

Verified against the live Actors on 2026-11-13. Prices are Apify Store pay-per-event prices on the FREE tier; paid Apify plans are 2.5 to 20 percent lower. Only charged records are billed; every run also charges one start event per started GB of memory.

All Actors belong to Apify user `muhammadafzal`. On the package-tracking MCP server, the tool name is the Actor ID with `/` replaced by `--`, for example `muhammadafzal--dhl-package-tracking-scraper`. Some apps add a server prefix or turn hyphens into underscores; match on the Actor slug. Each tool returns the run (`runId`, `status`, `storages.datasets.default.id`, `storages.keyValueStores.default.id`), not the rows. If a tool response already includes dataset items, use them; otherwise fetch them with `get-dataset-items`.

Run options (`memory`, `timeout`, `maxTotalChargeUsd`) go in the tool's `callOptions`, never inside `input`.

## Overview

| Actor | Tracks | Numbers per run | Per record | Start fee | Summary record |
|---|---|---|---|---|---|
| `dhl-package-tracking-scraper` | DHL Express, Parcel, eCommerce | 10 | USD 0.020 | USD 0.00005 | `SUMMARY` |
| `fedex-package-tracking-scraper` | FedEx | 30 | USD 0.020 | USD 0.005 | `SUMMARY` |
| `ups-package-tracking-scraper` | UPS | 25 | USD 0.030 | USD 0.005 | `OUTPUT` |
| `17track-package-tracking-scraper` | USPS and 1,000+ postal and cross-border carriers | 10 | USD 0.039 | USD 0.005 | `OUTPUT` |
| `multi-carrier-package-tracking` | FedEx International Connect, USPS (best effort), UPS summary | 25 | USD 0.040 | USD 0.005 | `OUTPUT` |
| `amazon-shipping-tracking-scraper` | Amazon Shipping and Multi-Channel Fulfillment (TBA, UK and more) | 200 | USD 0.015 | USD 0.005 | `OUTPUT` |
| `gofo-package-tracking-scraper` | GOFO and GOFO Express (formerly Cirro) | 1,000 | USD 0.010 | USD 0.005 | `OUTPUT` |
| `spx-express-package-tracking-scraper` | SPX Express (Shopee Express) in PH, MY, ID, VN, SG, BR | 500 | USD 0.005 | USD 0.005 | `OUTPUT` |
| `maersk-container-tracking-scraper` | Maersk containers, bills of lading, bookings | 20 (see section) | USD 0.008 | USD 0.005 | `OUTPUT` |

## Shared rules

- Not-found, blocked, invalid and failed numbers never produce a dataset row, with one exception: the Amazon Actor writes free `NOT_FOUND` and `HISTORY_EXPIRED` rows. Everything else is counted and explained in the key-value record named above.
- Read the key-value record whatever the run status was. Some Actors end `FAILED` when every number was blocked or failed, and the record still explains each number.
- Send only the fields listed below in `input`. Every Actor handles its own network routing internally, so no other fields are needed.

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
- `accessMode`: leave at `auto`, which reads DHL's public tracking site.
- `recipientPostalCode` (string) only with exactly one number, when DHL asks for destination verification.
- `language` is a 2-letter ISO code, `requesterCountryCode` a 2-letter country code.

Dataset row (camelCase): `trackingNumber`, `shipmentId`, `service` (`express`, `ecommerce`, `parcel-de`, `dsc`…), `productName`, `statusCode` (`pre-transit` | `transit` | `delivered` | `failure` | `unknown`), `status` (DHL text such as `DELIVERED`, `OUT FOR DELIVERY`), `description`, `statusTimestamp`, `estimatedDelivery`, `originCountryCode`, `originPostalCode`, `originCity`, `destinationCountryCode`, `destinationPostalCode`, `destinationCity`, `currentLocation`, `totalPieces`, `events[]` of `{timestamp, statusCode, status, description, location}` (newest first), `serviceUrl`, `sourceUrl`, `retrievedAt`, `isDemoData`, `raw`.

Key-value record `SUMMARY`: `outcome` (`data` | `demo` | `empty` | `blocked` | `failed` | `rejected`), `requested`, `delivered`, `demoRecords`, `empty`, `blocked`, `failed`, `notStarted`, `chargeLimitReached`, `accessRoute`, `warnings` (strings formatted `"<number>: <message>"`), `finishedAt`. Run status is SUCCEEDED even for empty or rejected runs.

---

## FedEx: `fedex-package-tracking-scraper`

Price: USD 0.020 per record (GOLD 0.017), start USD 0.005 per started GB (default memory 1 GB, so one event). Max 30 numbers per run. About 6 to 10 s per number; set `callOptions.timeout` to 900 for more than 15 numbers.

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

Price: USD 0.030 per record (GOLD 0.022), start USD 0.005 per started GB (default memory 1 GB, one event). Max 25 numbers per run. 10 to 60 s per number, plus retries; the default run timeout is 900 s, which covers a full run of 25.

Input:
```json
{
  "trackingNumbers": ["1Z999AA10123456784"],
  "locale": "en_US",
  "requestTimeoutSecs": 45
}
```
- `locale` is `language_COUNTRY`, for example `en_GB`; it selects the language and country version of the UPS tracking page.
- Accepts UPS package numbers and InfoNotice numbers, 7 to 34 letters or digits each.

Dataset row (camelCase): `trackingNumber`, `carrier` ("UPS"), `status` (`pre_advice` | `accepted` | `in_transit` | `out_for_delivery` | `delayed` | `delivery_attempted` | `ready_for_pickup` | `delivered` | `returned` | `exception` | `unknown`), `statusText`, `terminal`, `delivered`, `delayed`, `estimatedDelivery` (free text such as "Friday, October 23 by 7:00 P.M."), `lastEvent` (`{text, location}` or null), `deliveredTo`, `receivedBy`, `sourceUrl`, `scrapedAt`. No event history: UPS's public page shows only the latest update, and no Actor in this plugin returns the full UPS scan list.

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
- `carrier`: 0 for auto-detect; when 17TRACK reports several possible carriers the Actor tries up to two likely ones by itself. Otherwise one 17TRACK carrier code for the whole run (see carrier-formats.md).
- `language`: `en`, `de`, `es`, `fr`, `it`, `nl`, `pl`, `pt`, `tr`.

Dataset row (camelCase): `trackingNumber`, `displayTrackingNumber`, `carrierIds[]`, `carriers[]`, `status` (text), `subStatus`, `statusDescription`, `statusCategory` (`not_found` | `info_received` | `in_transit` | `pickup` | `out_for_delivery` | `undelivered` | `delivered` | `alert` | `expired` | `unknown`), `daysInTransit`, `originCountry`, `destinationCountry`, `estimatedDelivery`, `latestEvent` (`{timestamp, description, location, carrier}`), `events[]` (same shape), `sourceUrl`, `retrievedAt`, `isDemoData`, `raw`.

Key-value record `OUTPUT`: `outcome` (`DATA` | `DEMO` | `EMPTY` | `BLOCKED` | `FAILED` | `REJECTED` | `LIMIT_REACHED`), `requested`, `attempted`, `delivered`, `empty`, `blocked`, `failed`, `skipped`, `invalidInput`, `stopReason` (`completed` | `invalid_input` | `item_limit` | `charge_limit` | `runtime_limit` | `source_failure` | `free_plan_limit` | `execution_error`), `warnings[]` of `{trackingNumber (masked), category (invalid_input | empty | blocked | failed), message}`. The run ends FAILED when the outcome is BLOCKED or FAILED; still read OUTPUT.

Behaviour: the first blocked or failed number stops the batch and the remaining numbers are `skipped` (not charged). Rerun skipped numbers in a new run. Public demo number `7777777770` returns a labelled demo record.

Data-quality rule: `latestEvent` and `statusDescription` are reliable. The `events[]` list can arrive with the timestamp and description concatenated into one string (for example a `timestamp` of `2022-03-25 19:57CHINA,WUHAN, Processed…`). Only display `events[]` when every `timestamp` parses as a date on its own; otherwise show `latestEvent` and say the full history is available on the 17TRACK page at `sourceUrl`.

---

## Multi-carrier: `multi-carrier-package-tracking`

Price: USD 0.040 per record (GOLD 0.032), start USD 0.005. Max 25 numbers per run. Default run timeout 10 minutes; numbers not reached are counted in `notChecked`.

What each carrier returns (coverage as of 2026-11-10):
- **FedEx International Connect** (`KOT…`/`PRO…`): full shipment history. This is the only Actor that tracks these numbers.
- **USPS**: best effort, read from USPS's own tracking page with the scan history. USPS holds some automated lookups, especially several in a row; after two USPS numbers in a row get no answer, the rest of that run's USPS numbers are reported `BLOCKED` at once. Not charged when blocked or failed. Send at most 3 USPS numbers per run.
- **UPS**: the summary only (status, latest update, delivered-to or estimate, service). `events[]` holds just the latest update. The UPS Actor returns the same detail for less, so use that for UPS.
- **Standard FedEx** (12 or 15 digits): reported `BLOCKED` without a lookup. Use the FedEx Actor.

Input:
```json
{
  "trackingNumbers": ["KOT1234AB12345678901"],
  "carrier": "FedEx",
  "includeDetailedEvents": true,
  "requestTimeoutSecs": 45
}
```
- `carrier`: `auto` | `USPS` | `UPS` | `FedEx`. Give the carrier explicitly; `auto` rejects unrecognized formats as `ambiguous_carrier`.
- `requestTimeoutSecs`: 20 to 90, per USPS or FedEx International Connect page.

Dataset row (camelCase): `trackingNumber`, `carrier`, `status` (same enum as the UPS Actor), `statusText`, `terminal`, `delivered`, `delayed`, `estimatedDelivery`, `actualDelivery` (carrier free text such as `Tuesday, 10/21 at 2:41 P.M.`), `service`, `origin` and `destination` (`{city, stateOrProvince, postalCode, country}` or null), `lastEvent` (`{timestamp, status, description, location}`), `events[]` (same shape, carrier order), `sourceUrl`, `scrapedAt`.

Key-value record `OUTPUT`: `outcome` (`DATA` | `EMPTY` | `BLOCKED` | `REJECTED` | `FAILED`), counts as UPS plus `requested` and `notChecked`, `warnings[]` of `{trackingNumber (masked), carrier, category (not_found | invalid_input | ambiguous_carrier | blocked | failed | charge_limit), message}`, `stopReason` (`completed` | `charge_limit` | `runtime_limit` | `runtime_guard` | `provider_unavailable` | `platform_shutdown` | `runtime_error` | `invalid_input`), `effectiveLimit`. The run ends FAILED when nothing was delivered because every number was rejected, blocked or failed; still read OUTPUT.

Event timestamps are carrier free text, not ISO; a `status` like "A late flight has caused a delay…" inside `events[]` marks a delay.

---

## Amazon Shipping: `amazon-shipping-tracking-scraper`

Price: USD 0.015 per package found with live status (GOLD 0.012), start USD 0.005. `NOT_FOUND` and `HISTORY_EXPIRED` rows are free. Max 200 numbers per run; 200 numbers finish in under 3 minutes (default run timeout 300 s).

Tracks parcels shipped **with** Amazon by other stores: Amazon Shipping labels and Amazon Multi-Channel Fulfillment orders (Shopify, WooCommerce, eBay, own site). Orders placed on Amazon's own retail sites are only visible in the buyer's Your Orders page after sign-in; the public tracker answers `NOT_FOUND` for them.

Input:
```json
{
  "trackingNumbers": ["TBA335250058574", "UK4889319345"],
  "trackerCountry": "auto",
  "includeScanHistory": true
}
```
- `trackerCountry`: `auto` | `US` | `GB` | `IN` | `FR` | `IT` | `ES`. Leave at `auto`: TBA, TBC, TBM and numeric IDs go to the US tracker, UK/GB numbers to the UK tracker, IT, ES and FR prefixes to theirs, and Amazon's own hint is followed when a parcel belongs to another country.
- `includeScanHistory: false` gives a status-only row at the same price.

Dataset row (camelCase), one per requested number: `trackingNumber`, `lookupStatus` (`FOUND` | `HISTORY_EXPIRED` | `NOT_FOUND`), `status` (`LABEL_CREATED` | `IN_TRANSIT` | `OUT_FOR_DELIVERY` | `AVAILABLE_FOR_PICKUP` | `DELIVERED` | `DELIVERY_FAILED` | `RETURNING` | `RETURNED` | `CANCELLED` | `EXCEPTION` | `UNKNOWN`), `statusLabel`, `rawStatus`, `trackingStatusCode`, `statusTags[]`, `isDelivered`, `deliveredAt`, `deliveryLocation` (for example "Delivered to doorstep"), `expectedDeliveryAt`, `promisedDeliveryAt`, `isDelayed`, `labelCreatedAt`, `pickedUpAt`, `shipperName`, `service` (Amazon Shipping or Amazon Multi-Channel Fulfillment), `lastMileCarrier`, `lastMileByAmazon`, `shipmentDirection` (`FORWARD` | `RETURN`), `proofOfDeliveryPhoto` (boolean only), `lastEvent`, `lastLocation`, `eventCount`, `events[]` of `{eventCode, description, occurredAt, city, stateProvince, countryCode, postalCode, location, detailCode}` (**oldest first**), `trackerCountry`, `trackingUrl`, `errorCode`, `message`, `checkedAt`. All times are ISO 8601 UTC.

Key-value record `OUTPUT`: `outcome` (`DATA` | `EMPTY` | `PARTIAL` | `BLOCKED` | `FAILED` | `REJECTED`), `message`, `requestedCount`, `validCount`, `invalidTrackingNumbers[]`, `duplicatesRemoved`, `delivered` (`{total, found, historyExpired, notFound}`), `chargedEvents`, `chargeLimitReached`, `blockedTrackingNumbers[]`, `failedTrackingNumbers[]`, `skippedTrackingNumbers[]`, `stopReason`, `warnings[]`. The run ends FAILED when the outcome is BLOCKED or FAILED; still read OUTPUT.

Data-quality rules: Amazon serves scan history for 45 days. A `HISTORY_EXPIRED` row carries only a final summary (often `deliveredAt`); report it as "Delivered on <date>, scan history no longer available" when `deliveredAt` is set, else `expired`. For a `NOT_FOUND` row, show the `message` and, if the user bought the item on Amazon itself, say that those orders can only be tracked in Your Orders. Reverse `events[]` before display so the newest scan comes first.

---

## GOFO: `gofo-package-tracking-scraper`

Price: USD 0.010 per shipment found (GOLD 0.008), start USD 0.005. Max 1,000 numbers per run; GOFO answers 100 numbers per request, so 100 numbers take a few seconds. Default run timeout 90 s; set `callOptions.timeout` to 300 for more than 500 numbers.

Tracks GOFO and GOFO Express parcels (GOFO was formerly Cirro). Accepts waybill numbers (`GFUS…`) and the `CR…` reference number GOFO lists for the same parcel. Numbers that belong to GOFO's Canada, France, Netherlands, Italy or Spain sites are followed there automatically.

Input:
```json
{
  "trackingNumbers": ["GFUS01070717185345"],
  "includeEvents": true
}
```
- `includeEvents: false` returns the current status and last scan only.

Dataset row (camelCase): `trackingNumber`, `waybillNumber`, `referenceNumber`, `region` (`us` | `ca` | `fr` | `nl` | `it` | `es`), `status` (GOFO text: `Processing`, `Transit`, `Delivered`, `Alert`, `Returned`), `statusCategory` (`label_created` | `in_transit` | `out_for_delivery` | `delivered` | `exception` | `returned` | `unknown`), `isDelivered`, `deliveredAt`, `lastEventDescription`, `lastEventTimeUtc`, `lastEventLocalTime`, `lastEventLocation`, `estimatedDelivery`, `estimatedDeliveryEnd`, `labelCreatedAt`, `transitDays`, `transitBusinessDays`, `originCountry`, `destinationCountry`, `service`, `weight` (unit not stated by GOFO), `eventCount`, `events[]` of `{timeUtc, localTime, timeZone, description, code, location, city, state}` (newest first), `sourceUrl`, `retrievedAt`.

Key-value record `OUTPUT`: `outcome` (`data` | `partial` | `empty` | `blocked` | `failed` | `rejected`), `message`, `requestedNumbers`, `found`, `delivered`, `resultEventsCharged`, `notFound[]` (plain numbers), `failed[]`, `failureMessages`, `invalidNumbers[]`, `duplicateInputs`, `duplicateShipments`, `chargeLimitReached`, `stoppedBeforeTimeout`.

Behaviour: a waybill number and its `CR…` reference are the same parcel; sending both saves one row and charges once. Match a requested number against both `waybillNumber` and `referenceNumber` when reconciling. Show `lastEventLocalTime` with the location, since it is the clock time where the scan happened.

---

## SPX Express: `spx-express-package-tracking-scraper`

Price: USD 0.005 per parcel found (GOLD 0.004), start USD 0.005. Max 500 numbers per run; a batch of 37 finished in about 7 s in testing. Vietnam and Singapore lookups take 2 to 7 s each. Default run timeout 600 s.

Tracks SPX Express (Shopee Express) parcels in the Philippines, Malaysia, Indonesia, Vietnam, Singapore and Brazil. Numbers from several countries can share one run. Thailand and Taiwan numbers are not supported and are reported as `unsupported_country` without a charge.

Input:
```json
{
  "trackingNumbers": ["SPXID062930295622", "PH250040584072U"],
  "country": "auto"
}
```
- `country`: `auto` | `PH` | `MY` | `ID` | `BR` | `VN` | `SG`. Leave at `auto`, which reads the country from the prefix (`SPXPH`/`PH`, `SPXMY`/`MY`, `SPXID`, `SPXVN`, `SPXSG`, `BR`). Set it only when the numbers carry no country prefix.

Dataset row (camelCase): `trackingNumber`, `country`, `countryName`, `status` (`DELIVERED` | `OUT_FOR_DELIVERY` | `IN_TRANSIT` | `PREPARING_TO_SHIP` | `DELIVERY_UNSUCCESSFUL` | `RETURNED_TO_SENDER` | `LOST` | `OTHER`), `statusLabel`, `statusDescription`, `milestoneCode`, `milestoneName`, `latestEventCode`, `latestEventTime`, `lastKnownLocation` (`{name, address, latitude, longitude}` or null), `isDelivered`, `deliveredAt`, `shippedAt`, `transitDays`, `failedDeliveryAttempts`, `lastFailureReason`, `slsTrackingNumber`, `clientOrderId`, `isShopeeMarketplaceOrder`, `isInstantOrder`, `eventCount`, `events[]` of `{time, timestamp, code, name, description, milestoneCode, milestoneName, reasonCode, reason, location, nextLocation, shownOnTrackingPage}` (newest first), `trackingUrl`, `scrapedAt`. Times are ISO 8601 UTC.

Key-value record `OUTPUT`: `outcome` (`data` | `empty` | `blocked` | `failed` | `not_started` | `rejected`), `requested`, `delivered`, `noEvents`, `notFound`, `unsupportedCountry`, `invalidFormat`, `duplicates`, `blocked`, `failed`, `notStarted`, `stopReason`, `warnings[]`, and `numbers[]` with one entry per requested number: `{trackingNumber, country, outcome (data | not_found | no_events | unsupported_country | invalid_format | duplicate | blocked | failed | not_started), message}`. Numbers are not masked here, so reconcile directly from `numbers[]`.

Behaviour: `no_events` means SPX knows the number but has no scans yet, or removed the history. Show `clientOrderId` only when the user asks for the order ID. US SpeedX numbers also start with `SPX` but are a different company; they come back `invalid_format`.

---

## Maersk: `maersk-container-tracking-scraper`

Price: USD 0.008 per shipment found (GOLD 0.0064), start USD 0.005. The Actor accepts 100 numbers per run, and free Apify plans are limited to 20 per run, so send at most 20 per run. A number with results takes 15 to 30 s; two are checked at a time. A number without results takes about 3 minutes, because Maersk's page is checked three times before the number is reported not found. Default run timeout 1,800 s.

Tracks Maersk ocean shipments by container number (4 letters and 7 digits, for example `MSKU0078257`), by 9-character bill of lading or booking number, or by 7-digit reference. Other ocean carriers, air, LCL and parcel numbers are not supported.

Input:
```json
{
  "trackingNumbers": ["MSKU0078257", "258765432"]
}
```

Dataset row (camelCase), one per number with results; a bill of lading with several containers is one row: `trackingNumber`, `numberType` (`container` | `bill_of_lading` | `reference`), `shipmentStatus` (Maersk's name of the latest actual event: `Gate in`, `Load`, `Vessel departure`, `Vessel arrival`, `Discharge`, `Gate out for delivery`, `Empty container return`…), `latestEvent` and `nextEvent` (`{name, vessel, voyage, location, terminal, dateTime, isPlanned, containerNumber}`; `nextEvent` is null when nothing is planned), `origin` and `destination` (`{city, country, countryCode}`), `departureDate`, `departureTerminal`, `arrivalDate` (Maersk's ETA at the port of discharge until the vessel arrives, `YYYY-MM-DD`), `arrivalTerminal`, `containerCount`, `containers[]` of `{containerNumber, containerType, departure, arrival, lastUpdated, nextUpdateExpected, completedEventsCount, latestEvent, plannedEvents[]}`, `trackingUrl`, `checkedAt`.

Event times are local times at the event's location with no time zone (`YYYY-MM-DDTHH:mm`). Maersk's public page lists the latest event and the planned events still ahead; earlier completed events are only counted in `completedEventsCount`, so there is no full history to show.

Key-value record `OUTPUT`: `outcome` (`data` | `partial` | `empty` | `blocked` | `failed` | `rejected`), `message`, `requestedNumbers`, `delivered`, `resultEventsCharged`, `notFound[]`, `cancelled[]` (shipments Maersk lists as cancelled; no row, no charge), `failed[]`, `notLookedUp[]`, `noResultsUnconfirmed[]` (the run ended before the last recheck; rerun these), `invalidNumbers[]`, `containerNumbersWithBadCheckDigit[]` (usually a typo), `freePlanLimit`, `numbersSkippedByFreePlan`, `chargeLimitReached`, `stopReason`. Numbers are not masked.

---

## Cost formula

cost = runs × start fee + charged records × per-record price.

Examples on the FREE tier: 1 UPS number ≈ USD 0.035; 10 DHL numbers ≈ USD 0.20; 30 FedEx numbers ≈ USD 0.605; 25 UPS numbers ≈ USD 0.755; 10 17TRACK numbers ≈ USD 0.395; 100 Amazon Shipping packages ≈ USD 1.505; 500 GOFO shipments ≈ USD 5.005; 500 SPX parcels ≈ USD 2.505; 20 Maersk containers ≈ USD 0.165; 100 mixed numbers (40 FedEx, 30 UPS, 20 DHL, 10 USPS via 17TRACK) ≈ USD 0.805 + 0.905 + 0.40 + 0.395 ≈ USD 2.51.

Quote costs as "about", because not-found numbers are not charged and the user's Apify plan tier may be discounted.
