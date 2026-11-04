# Normalized status vocabulary

Every result is reported with one of these normalized statuses, in addition to the carrier's own wording.

| Normalized | Meaning | Shown as |
|---|---|---|
| `pre_transit` | label created, carrier has not scanned the parcel yet | Label created |
| `in_transit` | moving through the network | In transit |
| `out_for_delivery` | on the vehicle for delivery today | Out for delivery |
| `delivered` | delivered or left at a location | Delivered |
| `delivery_attempted` | attempt failed, redelivery or pickup pending | Delivery attempted |
| `ready_for_pickup` | held at a pickup point, locker or facility | Ready for pickup |
| `delayed` | still moving but behind the promised date | Delayed |
| `exception` | problem that needs action: address issue, customs hold, damaged, refused | Exception |
| `returned` | returning or returned to sender | Returned |
| `not_found` | carrier has no record for this number | Not found |
| `blocked` | carrier refused the lookup; not charged | Lookup blocked, retry |
| `expired` | too old for the carrier's public history | Expired |
| `unknown` | carrier returned no usable status | Unknown |

## Mapping from each Actor

**DHL** `statusCode`: `pre-transit` → pre_transit; `transit` → in_transit, or out_for_delivery when `status` contains OUT FOR DELIVERY, or ready_for_pickup when it contains PICKUP, HELD or AVAILABLE FOR COLLECTION; `delivered` → delivered; `failure` → exception, or delivery_attempted when `status` contains ATTEMPT or NOT DELIVERED, or returned when it contains RETURN; `unknown` → pre_transit when `status` is LABEL CREATED, else unknown.

**FedEx** `status_code`: `DL` → delivered; `IT`, `DP`, `AR`, `AF`, `PU` → in_transit; `OD` → out_for_delivery; `OC`, `IN` → pre_transit; `HL` → ready_for_pickup; `DE`, `SE`, `CD` → exception; `DY` → delayed; `RS`, `RR` → returned; `CA` → exception (cancelled). When `status_code` is null, classify from the `status` text with the same words. Placeholder rows (see actor-contracts.md) → unknown.

**UPS Actor and multi-carrier** `status`: `pre_advice` → pre_transit; `accepted`, `in_transit` → in_transit; `out_for_delivery` → out_for_delivery; `delayed` → delayed; `delivery_attempted` → delivery_attempted; `ready_for_pickup` → ready_for_pickup; `delivered` → delivered; `returned` → returned; `exception` → exception; `unknown` → unknown. If `delayed` is true and the status is in_transit, show delayed.

**17TRACK** `statusCategory`: `info_received` → pre_transit; `in_transit` → in_transit; `pickup` → ready_for_pickup; `out_for_delivery` → out_for_delivery; `undelivered` → delivery_attempted; `delivered` → delivered; `alert` → exception; `expired` → expired; `not_found` → not_found; `unknown` → unknown. When `statusCategory` is unknown but `status` text is in another language, classify from the text if you can, else unknown.

**Key-value warnings** (numbers without a row): category `not_found` or `empty` → not_found; `blocked` → blocked; `invalid_input` → report "invalid number" and do not count it as a lookup; `failed` → unknown with the message; `charge_limit`, `skipped`, `notStarted` → "not attempted (limit reached)", offer to rerun.

## Shared result schema

```
trackingNumber      string   as requested (normalized)
carrier             string   DHL | FedEx | UPS | USPS | <17TRACK carrier name> | unknown
status              string   normalized value from the table above
statusText          string   carrier wording
lastEvent           object   { time, description, location } or null
estimatedDelivery   string   carrier text or null
actualDelivery      string   carrier text or null
origin              string   "City, Region, Country" or country only, or null
destination         string   same
service             string   or null
events              array    newest first, only when reliable
sourceUrl           string   carrier or 17TRACK page
actorUsed           string   Actor slug
note                string   data-quality or warning message, or null
```
