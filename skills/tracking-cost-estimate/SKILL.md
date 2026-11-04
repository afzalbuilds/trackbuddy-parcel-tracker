---
name: tracking-cost-estimate
description: Estimate how much a package-tracking batch will cost on Apify before running it. Use when the user asks "how much will it cost to track", "what does tracking cost", "price per tracking number", "is it free", "how is this billed", "which tracker is cheapest", or wants to compare the cost of tracking DHL, FedEx, UPS, USPS or 17TRACK numbers, or asks about Apify credits, plans or charges for this plugin.
---

# Tracking cost estimate

Prices are in `../track-package/references/actor-contracts.md` (FREE-tier Apify Store prices, verified 2026-11-03). Use that file as the single source; do not quote other numbers.

## How billing works (explain in two or three sentences when asked)

The plugin is free. Each tracking Actor is a pay-per-event Actor on Apify Store, billed to the user's own Apify account: one event per record delivered to the dataset, plus a small start event per run. Numbers that are not found, blocked or invalid are not charged as records. Free Apify accounts include monthly usage credit; paid plans get tiered discounts of up to 20 percent on the per-record price.

## Estimate

1. Count numbers per Actor using the routing rules in `../track-package/references/carrier-formats.md`.
2. Runs per Actor = ceil(count / per-run limit): DHL 10, 17TRACK 10, UPS 25, multi-carrier 25, FedEx 30.
3. Cost per Actor = runs × start fee + count × per-record price.
4. Total = sum. Present a small table (Actor, numbers, runs, per record, subtotal) and the total as "about USD X". Mention the assumption that every number returns data; the real charge is lower when some are not found.
5. If the user asks which option is cheaper for UPS: the UPS Actor (USD 0.030, latest status only) versus multi-carrier (USD 0.040, full scan history). For USPS and most international postal numbers, 17TRACK is the only route (USD 0.039).
6. Add the rough time: about 10 to 30 seconds per number inside a run, runs for different Actors in parallel.

Do not run any Actor from this skill. If the user then says to go ahead, hand over to `track-package` or `bulk-tracking-report`.
