---
name: track-container
description: Track Maersk ocean freight by container number, bill of lading or booking number and report status, vessel, next planned event and arrival date (ETA). Use when the user says "track container", "container tracking", "Maersk tracking", "where is my container", "track bill of lading", "track B/L", "track booking", "when does my container arrive", "vessel ETA", or pastes container numbers such as MSKU0078257 or MAEU1234567, or a 9-character Maersk B/L or booking number.
---

# Track a Maersk container

You track Maersk ocean shipments by running the `maersk-container-tracking-scraper` Actor on Apify through the **package-tracking** MCP server (some apps call it a connector). The tool name contains the Actor slug, `muhammadafzal--maersk-container-tracking-scraper`, possibly with a server prefix or underscores in place of hyphens. The server also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`.

Read the Maersk section of `../track-package/references/actor-contracts.md` before the first call, and the "Ocean freight" section of `../track-package/references/carrier-formats.md` for which numbers to accept. Parcels and courier shipments belong to the `track-package` skill.

## Steps

1. **Collect the numbers.** Container numbers are 4 letters and 7 digits; bill of lading and booking numbers are 9 characters; Maersk tracking links work too. Strip spaces and hyphens, uppercase, drop duplicates. If a container prefix is not one of Maersk's (MAEU, MSKU, MRKU, MRSU, MNBU) and the user has not said the cargo ships with Maersk, say that only Maersk shipments can be tracked and ask whether to try it anyway.

2. **Batch and estimate.** Send at most 20 numbers per run. Cost is about USD 0.008 per shipment found plus USD 0.005 per run. For more than 10 numbers, or an estimate above USD 0.50, show the estimate and the expected time, then wait for the go-ahead. Time: 15 to 30 seconds per number with results, two at a time; a number Maersk has no results for takes about 3 minutes to confirm.

3. **Run** the Actor with `{"trackingNumbers": [...]}` and nothing else in `input`.

4. **Wait.** Call `get-actor-run` with the `runId` and `waitSecs: 45` until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`. A run with a not-found number can take several minutes; tell the user you are waiting. Never report a run as done while it is still running.

5. **Fetch** the dataset items and the `OUTPUT` key-value record, whatever the run status was.

6. **Reconcile.** Every requested number appears in the answer once. Numbers without a row are explained in `OUTPUT`: `notFound` (no public Maersk tracking; typo, not yet registered, or not carried by Maersk), `cancelled` (Maersk lists the shipment as cancelled), `containerNumbersWithBadCheckDigit` (almost always a typo; ask the user to recheck), `invalidNumbers`, `failed`, and `noResultsUnconfirmed`, `notLookedUp` or `numbersSkippedByFreePlan` (not finished; offer to rerun).

## Answer format

One table: Number, Type (container, B/L or booking), Status (Maersk's event name), Latest event (local time, location, terminal, vessel and voyage), Next planned event (with its estimated time), Route (origin → destination), Arrival date, Notes. Put delayed and cancelled shipments first. For a bill of lading with several containers, add one sub-row per container with its type and latest event.

Then one line: "Apify cost: about USD X for N shipments in M runs, billed to your Apify account."

Rules:
- `arrivalDate` and every planned event are Maersk's estimates; call them "estimated" until the event has happened.
- Event times are local to the event's location and carry no time zone. Show them as given.
- Maersk's public page lists only the latest event and the planned events still ahead. If the user asks for the full history, say that earlier events are counted (`completedEventsCount`) but not listed, and give `trackingUrl`.
- When the user gives a deadline ("which arrive after 15 December"), compare it with `arrivalDate` and list the late ones first.
- The Actor reads no customer names, addresses or documents; do not imply that it can.
