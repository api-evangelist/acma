---
name: Find every radiocommunications licence held by an organisation
description: >-
  Resolve an Australian organisation to its ACMA client number, then list every
  radiocommunications licence it holds and each licence's registered devices — the
  standard due-diligence traversal over the Register of Radiocommunications Licences.
api: openapi/acma-spectrum-licensing-openapi.yml
operations:
  - clientSearchJSON
  - licenceSearchJSON
  - registrationSearchJSON
  - siteSearchJSON
generated: '2026-07-25'
method: generated
---

# Find every licence held by an organisation

The ACMA Spectrum Licensing API has no join and no expand. You resolve the licensee first,
then walk outward by re-querying with the id you just learned. Four calls, in this order.

## Before you start

- **No authentication.** No key, no header, no account. Call the endpoint directly:
  `https://api.acma.gov.au/SpectrumLicensingAPIOuterService/OuterService.svc`
- **Never assume you saw everything.** Every response caps at 2,000 records. Read
  `TOTAL_RESULT` off any record and compare it to what you received; if it is larger,
  re-issue with `offset=2000`, `offset=4000`, and so on.
- **An empty result is HTTP 200 with an empty array**, not a 404.

## Step 1 — resolve the organisation to a client number

Call `clientSearchJSON` with the organisation name in the path and the field narrowed:

```
GET /ClientSearchJSON/{name}?searchField=Org%20Name&searchOption=begins%20with&resultsLimit=25
```

`searchOption` accepts `matches`, `begins with` and `sounds like`. Prefer `begins with` for
company names — `matches` is exact and will miss "PTY LTD" variants, and `sounds like` is a
phonetic match that will pull in unrelated companies.

If you have the ABN or ACN instead, that is a stronger key: pass it as `searchText` with
`searchField=ABN` or `searchField=ACN`.

Take `CLIENT_NO` from the record whose `DISPLAY_NAME` you have confirmed. Treat it as a
string — this API returns it as a JSON number here and as a string elsewhere.

## Step 2 — list the licences under that client number

```
GET /LicenceSearchJSON?searchText={CLIENT_NO}&searchField=CLIENT_NO&sortBy=licence_no&resultsLimit=2000
```

`searchText` is a **query parameter, not a path segment**, on this operation — that is
deliberate, because licence numbers contain slashes (`11265050/1`).

Each row gives you `LICENCE_NO`, `LICENCE_CATEGORY`, `STATUS_TEXT`, `DATE_EXPIRY`,
`CALLSIGN`, `SHIP_NAME` and a `DETAILS_URL` pointing at the human register page. Page on
`TOTAL_RESULT` before you report a count — a large carrier will exceed one page.

## Step 3 — pull the registered devices for a licence

```
GET /RegistrationSearchJSON?searchText={LICENCE_NO}&searchField=LICENCE_NO&resultsLimit=2000
```

Returns `FREQ` (in Hz), `EMISSION_DESIG`, `DEVICE_TYPE_TEXT`, `EFL_ID` and `SITE_ID`.

## Step 4 — resolve each site

```
GET /SiteSearchJSON/{SITE_ID}?searchField=SITE_ID&resultsLimit=1
```

Gives `LONG_NAME`, `CITY`, `STATE`, `LATITUDE` and `LONGITUDE`, so the organisation's
licences can be plotted.

## Reporting rules

- Licence status lives in `STATUS_TEXT` (e.g. `Granted`). Do not describe a licence as
  active without it.
- Frequencies come back in **Hz** on registration and assignment rows (`1937500000.0`) but in
  **MHz** on the 400 MHz register (`462.1125`). State the unit every time.
- Dates are local ISO datetimes with no timezone (`2027-05-31T00:00:00`).
- ACMA's licence terms **forbid** using licensee personal information for unsolicited
  telemarketing, spam or mail advertising, and forbid redistributing a natural person's
  details in a derivative work. Attribute any published output as
  "Based on Australian Communications and Media Authority information".

## When to use the bulk download instead

If you need every licence in Australia rather than one organisation's, do not page this API.
ACMA publishes a complete same-day extract as a zip at
<https://www.acma.gov.au/register-radiocommunication-licences-rrl#/data-download>; the
2,000-record cap exists specifically to push that use case there.
