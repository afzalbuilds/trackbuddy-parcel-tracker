---
name: delivery-exceptions
description: Triage delivery problems and draft customer or carrier messages. Use when a package is delayed, late, stuck in transit, shows an exception, a failed delivery attempt, "held at customs", "address issue", "returned to sender", "delivered but not received", or when a customer-support, ecommerce or logistics user asks what to do about a problem shipment, how to reply to a "where is my order" ticket, or how to file a claim.
---

# Delivery exception triage

If the shipment has not been tracked in this conversation yet, run the `track-package` skill first so the triage is based on the carrier's current data, not on assumptions.

## 1. Classify the problem

| Situation | Signals | Typical cause |
|---|---|---|
| Not scanned yet | status `pre_transit`, label created, no scans for 24 to 48 h | shipper has not handed over, or the first scan is pending |
| Stalled in transit | last scan older than 3 business days (domestic) or 7 days (international), status still `in_transit` | missorted, left behind at a hub, awaiting a linehaul |
| Customs hold | 17TRACK or DHL text mentions customs, duties, clearance, "held", "retained" | duties unpaid, missing documents, inspection |
| Delivery attempted | `delivery_attempted`, "notice left", "InfoNotice", "door tag" | nobody home, no access, signature required |
| Ready for pickup | `ready_for_pickup`, held at location, access point, locker | redirected after attempts, or the recipient chose pickup |
| Address problem | exception text with address, "unable to locate", "incorrect address", UPS "corrected the postal code" | wrong or incomplete address |
| Damaged or lost | exception mentions damaged, missing contents, "unable to deliver", or no scans for 14+ days | handling damage or lost in network |
| Returned | `returned`, "return to sender", "refused" | unclaimed, refused, undeliverable |
| Delivered but missing | `delivered` with a delivered-to location but the recipient reports nothing | left in a safe place, delivered to a neighbour, mailroom, or misdelivery |

## 2. Recommend the next action

- **Not scanned yet**: wait until 48 h after the label date, then ask the shipper to confirm handover. Do not open a carrier claim yet.
- **Stalled**: wait one more business day, then open a carrier trace: UPS and FedEx accept traces from the shipper; DHL from shipper or recipient; postal items (USPS, China Post, Royal Mail) via the origin post or the marketplace (AliExpress, Temu, Shein dispute windows apply).
- **Customs hold**: the recipient usually has to pay duties or supply ID on the carrier's site; give the tracking URL from the result.
- **Delivery attempted or ready for pickup**: tell the recipient where it is held, the pickup deadline (UPS and FedEx typically hold 5 to 7 days, DHL 7 days, USPS 15 days), and how to reschedule using the door tag or InfoNotice number.
- **Address problem**: correct the address with the carrier within 24 h (UPS Delivery Change, FedEx Delivery Manager, DHL On Demand Delivery); after that the parcel returns to sender.
- **Damaged or lost**: start a claim: UPS within 60 days, FedEx within 60 days (US) or 21 days (international), DHL Express within 30 days, USPS within 60 days. Say that windows vary by country and service and that the shipper usually files.
- **Returned**: confirm whether to reship or refund; check the return scan location.
- **Delivered but missing**: ask the recipient to check the delivered-to location, safe places, neighbours and the mailroom, wait 24 h (carriers sometimes scan early), then request proof of delivery from the carrier.

Give the carrier's official tracking URL from the result for every recommendation so the user can act on it.

Carrier-specific notes:
- **Amazon Shipping / Multi-Channel Fulfillment**: the merchant, not the recipient, opens the case with Amazon (Amazon Shipping support, or Seller Central for MCF). `deliveryLocation` says where the driver left the parcel and `proofOfDeliveryPhoto: true` means the merchant can request the photo from Amazon.
- **SPX Express (Shopee)**: `lastFailureReason` and `failedDeliveryAttempts` say why and how often delivery failed. Buyers resolve it in the Shopee app (return/refund window); sellers through Shopee Seller Centre.
- **GOFO**: contact GOFO support with the waybill number (`GF…`); the shipper files claims.
- **Maersk ocean freight**: a missed `arrivalDate`, a rolled vessel or a long gap before `Gate out for delivery` usually means port congestion, a customs hold or unpaid charges. The consignee or forwarder checks demurrage and detention free time and contacts Maersk customer service or the forwarder with the B/L number. Do not quote free-time days; they depend on the contract.

## 3. Draft the message when asked

Write in the user's voice, short and factual:
- To a customer: what the tracking shows (status, last scan time and place), what you are doing about it, what they can do, and when you will update them. Never promise a delivery date the carrier has not given.
- To a carrier or shipper: tracking number, service, ship date, last scan, the problem, the action requested (trace, redelivery, address correction, claim), and a contact.

Offer a one-line version for chat or SMS and a longer version for email. Do not invent order details; use placeholders in square brackets for anything you do not know.
