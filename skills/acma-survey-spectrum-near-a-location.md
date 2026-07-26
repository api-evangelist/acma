---
name: Survey licensed spectrum near a location
description: >-
  Given a latitude/longitude or a postcode range, find the nearest licensed transmitter
  sites and the frequency assignments around them — the site-selection and interference
  check an RF engineer runs before deploying equipment in Australia.
api: openapi/acma-spectrum-licensing-openapi.yml
operations:
  - spectrumLicenceSiteSearchByLocationJSON
  - assignmentRangeJSON
  - registrationSearchJSON
  - clientSearchJSON
  - showAccessAreaSearchJSON
generated: '2026-07-25'
method: generated
---

# Survey licensed spectrum near a location

Two entry points exist and they answer different questions. Pick deliberately.

- **By coordinates** — `spectrumLicenceSiteSearchByLocationJSON` returns the nearest sites
  with a real distance. Use it when you have a point.
- **By postcode + frequency + date range** — `assignmentRangeJSON` returns the assignments
  themselves. Use it when you have a band and an area.

## Before you start

No authentication. Every call caps at 2,000 records; read `TOTAL_RESULT` and page with
`strOffset`. Note the parameter names differ across this API: the search operations use
`offset`/`resultsLimit`, the operations in this skill mostly use `strOffset`/`strLimit`.

## Step 1 — nearest sites to a point

```
GET /SiteByLocationJSON/{latitude}/{longitude}?strLimit=25
```

Latitude is negative in Australia (`-35.238159`). Each row returns `SITE_ID`, `LONG_NAME`,
`CITY`, `STATE`, `POSTCODE`, coordinates and **`DISTANCE_KMS`** — the only distance the API
computes for you. Results are the closest N sites, so a large `strLimit` over a remote
coordinate will return sites hundreds of kilometres away; always report `DISTANCE_KMS`
rather than implying proximity.

## Step 2 — what is assigned in the band

```
GET /AssignmentRangeJSON?strLowerPostCode=2600&strUpperPostCode=2620
  &strLowerFreq=400000000&strUpperFreq=520000000
  &strLowerDate=01-01-2015&strUpperDate=01-01-2026
  &strLimit=2000&sortBy=frequency
```

Request dates are **DD-MM-YYYY** — the opposite convention to the ISO datetimes that come
back in responses. Frequencies on this operation are in **Hz**.

Each row gives `FREQ`, `BANDWIDTH`, `EMISSION_DESIG`, `OP_MODE`, `LICENCE_NO`, `CLIENT_NO`,
`SITE_ID` and `AUTHORISATION_DATE`. All range parameters are optional — omitting the postcode
pair widens the query to the whole country, which will be truncated at 2,000 records without
warning. Narrow first.

## Step 3 — attribute an assignment

- Licensee: `clientSearchJSON/{CLIENT_NO}?searchField=Client%20No.`
- Device detail: `registrationSearchJSON?searchText={LICENCE_NO}&searchField=LICENCE_NO`
- Access area for a spectrum licence: `showAccessAreaSearchJSON` with a code from 1 to 18.

## Correctness rules

- **Units are inconsistent across operations.** Assignment and registration rows carry `FREQ`
  in Hz; the 400 MHz register carries `ASSIGN_FREQ` in MHz. Convert explicitly, never infer.
- `AREA_ID` on assignment rows was null in every sampled record — do not rely on it to
  resolve the access area; use the licence's own area instead.
- The register reflects **licences**, not transmissions. A licensed assignment is not proof a
  transmitter is on air, and an absence in the register is not proof a band is clear —
  class-licensed and defence spectrum are not in this dataset.
- Data refreshes daily. There is no ETag or Last-Modified, so cache for no longer than a day.

## Bulk alternative

For a whole-of-Australia occupancy study, take the daily zip extract from
<https://www.acma.gov.au/register-radiocommunication-licences-rrl#/data-download> rather
than paging this API 100+ times.
