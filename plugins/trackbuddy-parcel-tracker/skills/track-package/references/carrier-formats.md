# Carrier detection and routing

Normalize first: trim, remove inner spaces and hyphens, uppercase. Then test the rules in order; the first match wins. "Actor" names the tool to call on the package-tracking MCP server.

| # | Pattern (regex) | Carrier | Actor | Notes |
|---|---|---|---|---|
| 1 | `^(KOT\|PRO)\d{4}[A-Z]{2}\d{11}$` | FedEx International Connect | multi-carrier-package-tracking (`carrier: "FedEx"`) | the FedEx Actor cannot track FIC numbers |
| 2 | `^1Z[A-Z0-9]{16}$` | UPS | ups-package-tracking-scraper | |
| 3 | `^TB[ACM]\d{12}$` | Amazon Shipping / Amazon Logistics (TBA US, TBC Canada, TBM Mexico) | amazon-shipping-tracking-scraper | works for Amazon Shipping and Multi-Channel Fulfillment parcels; orders bought on Amazon itself come back `NOT_FOUND` (free) and are only visible in Your Orders |
| 4 | `^UK\d{10}$` | Amazon Shipping UK | amazon-shipping-tracking-scraper | |
| 5 | `^GF[A-Z]{2}\d{10,}$` or `^CR\d{12}$` | GOFO / GOFO Express (formerly Cirro) | gofo-package-tracking-scraper | `GFUS…` is the waybill, `CR…` the reference for the same parcel |
| 6 | `^SPX(PH\|MY\|ID\|VN\|SG)[A-Z0-9]{8,}$` or `^(PH\|MY\|BR)\d{10,}[A-Z0-9]?$` | SPX Express (Shopee Express) | spx-express-package-tracking-scraper (`country: "auto"`) | `SPXTH…` (Thailand) and Taiwan numbers are not supported by this Actor; send those to 17TRACK |
| 7 | `^(MAEU\|MSKU\|MRKU\|MRSU\|MNBU)\d{7}$` | Maersk container | maersk-container-tracking-scraper | see "Ocean freight" below for other container prefixes and bills of lading |
| 8 | `^T\d{10}$` | UPS freight or Mail Innovations reference | ups-package-tracking-scraper | rare; confirm with the user |
| 9 | `^DT\d{12}$` | FedEx door tag | fedex-package-tracking-scraper | strip the DT prefix only if FedEx rejects it |
| 10 | `^\d{12}$` | FedEx (default) or UPS InfoNotice | fedex-package-tracking-scraper | if the user says "InfoNotice" or "UPS", use the UPS Actor |
| 11 | `^\d{15}$` | FedEx | fedex-package-tracking-scraper | |
| 12 | `^96\d{18,20}$` | FedEx SmartPost / Ground Economy | fedex-package-tracking-scraper | 20 or 22 digits starting 96 |
| 13 | `^(92\|93\|94\|95\|82)\d{18,22}$` | USPS (also UPS Mail Innovations, DHL eCommerce last mile) | 17track-package-tracking-scraper (`carrier: 0`) | fallback: multi-carrier with `carrier: "USPS"`, see below |
| 14 | `^[A-Z]{2}\d{9}US$` | USPS international | 17track-package-tracking-scraper | |
| 15 | `^\d{26}$` | USPS | 17track-package-tracking-scraper | |
| 16 | `^\d{10}$` | DHL Express waybill | dhl-package-tracking-scraper | 17TRACK's public demo number 7777777770 is also 10 digits; DHL's demo is the same number |
| 17 | `^(JD\|JJD)\d{14,20}$` or `^GM\d{16,20}$` or `^\d{20}$` starting with `00340434` | DHL eCommerce / DHL Paket / DHL Parcel | dhl-package-tracking-scraper | |
| 18 | `^[A-Z]{2}\d{9}[A-Z]{2}$` | Universal postal (China Post `…CN`, Royal Mail `…GB`, Deutsche Post `…DE`, Correos `…ES`, Japan Post `…JP`, and others) | 17track-package-tracking-scraper | includes `R…CN`, `L…CN`, `C…CN` registered and ePacket |
| 19 | `^YT\d{16}$` | YunExpress | 17track-package-tracking-scraper | common for AliExpress, Temu, Shein |
| 20 | `^LP\d{14,16}$` | Cainiao (AliExpress Standard Shipping) | 17track-package-tracking-scraper | |
| 21 | `^4PX\d{12,}$` or `^UA\d{9}YP$` or `^[A-Z]{2}\d{9}YP$` | 4PX / Yanwen / YunExpress postal | 17track-package-tracking-scraper | |
| 22 | `^JT\d{10,}$` or `^JNT\w+$` | J&T Express | 17track-package-tracking-scraper | |
| 23 | `^[A-Z]{3}U\d{7}$` | Ocean container of unknown carrier | see "Ocean freight" | |
| 24 | `^\d{9}$` | UPS reference or Maersk bill of lading / booking | ask | 9 digits is ambiguous; use the user's wording (container, B/L, booking, vessel, Maersk → Maersk; otherwise UPS) or ask |
| 25 | anything else, 4 to 80 alphanumerics | unknown | 17track-package-tracking-scraper (`carrier: 0`, auto-detect) | if 17TRACK reports "cannot auto-detect", ask the user for the carrier and rerun with its 17TRACK carrier code |

When the user explicitly names the carrier, route to that carrier's Actor even if the format does not match, except: standard FedEx never goes to multi-carrier, and FedEx International Connect never goes to the FedEx Actor. Words that name a carrier: "Amazon", "TBA", "MCF" → Amazon Shipping; "GOFO", "Cirro" → GOFO; "SPX", "Shopee", "Shopee Express" → SPX Express; "Maersk", "container", "bill of lading", "B/L", "booking" → Maersk.

## Ocean freight (Maersk)

The Maersk Actor is the only ocean-freight tracker in this plugin. It accepts:

- **Container numbers**: 4 letters and 7 digits. Maersk's own prefixes are MAEU, MSKU, MRKU, MRSU and MNBU, but a Maersk shipment can also travel in a container with another prefix, so when the user says the cargo ships with Maersk, send any container number.
- **Bill of lading or booking numbers**: 9 characters, letters or digits.
- **7-digit reference numbers**, only when the user says it is a Maersk reference.

A container number with a non-Maersk prefix and no mention of Maersk (rule 23): tell the user it looks like an ocean container, that only Maersk shipments can be tracked here, and offer to try it on Maersk. Do not send containers of MSC, CMA CGM, Hapag-Lloyd, COSCO, ONE, Evergreen or other lines unless the user asks to try; a miss costs nothing but takes about 3 minutes to confirm.

## USPS: which Actor

| Situation | Actor | Why |
|---|---|---|
| default for USPS numbers | 17track-package-tracking-scraper | dependable, USD 0.039 |
| 17TRACK blocked the number, or its event list is garbled and the user wants the full scan history | multi-carrier-package-tracking with `carrier: "USPS"`, at most 3 numbers per run | reads USPS's own page with the scan history (USD 0.040), but USPS holds some automated lookups; blocked numbers are free |

## UPS: one Actor

Use `ups-package-tracking-scraper` for every UPS number (USD 0.030). UPS's public page shows only the latest update, so no Actor returns the full UPS scan list; say so when the user asks for the UPS history and give the `sourceUrl` for the carrier page.

## 17TRACK carrier codes (for the `carrier` input when auto-detect fails)

Codes confirmed in the Actor's input schema: FedEx `100003`, DHL Paket `7041`. For any other carrier, ask the user to look up the numeric code on 17TRACK's carrier list (https://www.17track.net/en/carriers) and send it as `carrier`. One code applies to the whole run, so group numbers by carrier before using a code. Auto-detect (`carrier: 0`) handles the large majority of postal and e-commerce numbers and already retries the two most likely carriers for an ambiguous number, so only fall back to a code when the Actor reports that it cannot auto-detect.

17TRACK also aggregates many carriers that have a dedicated Actor here (GOFO, SPX, UPS, FedEx, DHL). Prefer the dedicated Actor, which is cheaper and more detailed, and use 17TRACK as the second try when the dedicated Actor reports the number as blocked or unsupported.
