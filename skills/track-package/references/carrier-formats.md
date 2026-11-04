# Carrier detection and routing

Normalize first: trim, remove inner spaces and hyphens, uppercase. Then test the rules in order; the first match wins. "Actor" names the connector tool to call.

| # | Pattern (regex) | Carrier | Actor | Notes |
|---|---|---|---|---|
| 1 | `^(KOT\|PRO)\d{4}[A-Z]{2}\d{11}$` | FedEx International Connect | multi-carrier-package-tracking (`carrier: "FedEx"`) | the FedEx Actor cannot track FIC numbers |
| 2 | `^1Z[A-Z0-9]{16}$` | UPS | ups-package-tracking-scraper | for full scan history use multi-carrier with `carrier: "UPS"` instead |
| 3 | `^T\d{10}$` or `^\d{9}$` | UPS freight or Mail Innovations reference | ups-package-tracking-scraper | rare; confirm with the user |
| 4 | `^DT\d{12}$` | FedEx door tag | fedex-package-tracking-scraper | strip the DT prefix only if FedEx rejects it |
| 5 | `^\d{12}$` | FedEx (default) or UPS InfoNotice | fedex-package-tracking-scraper | if the user says "InfoNotice" or "UPS", use the UPS Actor |
| 6 | `^\d{15}$` | FedEx | fedex-package-tracking-scraper | |
| 7 | `^96\d{18,20}$` | FedEx SmartPost / Ground Economy | fedex-package-tracking-scraper | 20 or 22 digits starting 96 |
| 8 | `^(92\|93\|94\|95\|82)\d{18,22}$` | USPS (also UPS Mail Innovations, DHL eCommerce last mile) | 17track-package-tracking-scraper (`carrier: 0`) | multi-carrier cannot do USPS |
| 9 | `^[A-Z]{2}\d{9}US$` | USPS international | 17track-package-tracking-scraper | |
| 10 | `^\d{26}$` | USPS | 17track-package-tracking-scraper | |
| 11 | `^\d{10}$` | DHL Express waybill | dhl-package-tracking-scraper | 17TRACK's public demo number 7777777770 is also 10 digits; DHL's demo is the same number |
| 12 | `^(JD\|JJD)\d{14,20}$` or `^GM\d{16,20}$` or `^\d{20}$` starting with `00340434` | DHL eCommerce / DHL Paket / DHL Parcel | dhl-package-tracking-scraper | |
| 13 | `^[A-Z]{2}\d{9}[A-Z]{2}$` | Universal postal (China Post `…CN`, Royal Mail `…GB`, Deutsche Post `…DE`, Correos `…ES`, Japan Post `…JP`, and others) | 17track-package-tracking-scraper | includes `R…CN`, `L…CN`, `C…CN` registered and ePacket |
| 14 | `^YT\d{16}$` | YunExpress | 17track-package-tracking-scraper | common for AliExpress, Temu, Shein |
| 15 | `^LP\d{14,16}$` | Cainiao (AliExpress Standard Shipping) | 17track-package-tracking-scraper | |
| 16 | `^4PX\d{12,}$` or `^UA\d{9}YP$` or `^[A-Z]{2}\d{9}YP$` | 4PX / Yanwen / YunExpress postal | 17track-package-tracking-scraper | |
| 17 | `^JT\d{10,}$` or `^JNT\w+$` | J&T Express | 17track-package-tracking-scraper | |
| 18 | `^(TBA\|TBC)\w{9,}$` | Amazon Logistics | none | tell the user Amazon TBA numbers are only visible in the Amazon order page |
| 19 | anything else, 4 to 80 alphanumerics | unknown | 17track-package-tracking-scraper (`carrier: 0`, auto-detect) | if 17TRACK reports "cannot auto-detect", ask the user for the carrier and rerun with its 17TRACK carrier code |

When the user explicitly names the carrier, route to that carrier's Actor even if the format does not match, except: USPS and standard FedEx never go to multi-carrier, and FedEx International Connect never goes to the FedEx Actor.

## 17TRACK carrier codes (for the `carrier` input when auto-detect fails)

Codes confirmed in the Actor's input schema: FedEx `100003`, DHL Paket `7041`. For any other carrier, ask the user to look up the numeric code on 17TRACK's carrier list (https://www.17track.net/en/carriers) and pass it as `carrier`. One code applies to the whole run, so group numbers by carrier before passing a code. Auto-detect (`carrier: 0`) handles the large majority of postal and e-commerce numbers, so only fall back to a code when the Actor reports that it cannot auto-detect.

## Choosing between UPS Actors

| User wants | Actor | Why |
|---|---|---|
| current status, ETA, delivered-to | ups-package-tracking-scraper | cheaper (USD 0.030) |
| every scan event, origin and destination, actual delivery timestamp | multi-carrier-package-tracking with `carrier: "UPS"` | the UPS Actor returns only the latest update (USD 0.040 per record) |
