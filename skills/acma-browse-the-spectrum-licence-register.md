---
name: Browse the spectrum licence register by category
description: >-
  Start from nothing and work down: list the licence categories and their counts, drill into
  a band, then pull the full 400 MHz band register record for a licence — the orientation
  path for an agent that does not yet know what it is looking for.
api: openapi/acma-spectrum-licensing-openapi.yml
operations:
  - spectrumLicencesCategoryListJSON
  - spectrumLicenceListJSON
  - spectrumLicence400MHzRegisterSearchJSON
  - licenceSearchJSON
generated: '2026-07-25'
method: generated
---

# Browse the spectrum licence register by category

This is the only zero-argument entry point in the API. Use it when you have no licence
number, no client number and no coordinate.

## Step 1 — list the categories

```
GET /CategoryListJSON
```

No parameters required. Returns every licence category with `LICENCE_TYPE`,
`LICENCE_CATEGORY_NAME` and `LIC_COUNT` — the count of licences in that category. This is the
map of the register: spectrum bands (`700 MHz Band`, `800 MHz Band`, `1800 MHz Band`, …)
alongside apparatus categories. Add `licenceType=Spectrum` to restrict to spectrum licences.

`LIC_COUNT` is the number that tells you whether the next step will fit in one page.

## Step 2 — drill into a category

```
GET /LicenceListJSON/{licenceCategory}?licenceType=Spectrum&strLimit=2000
```

The category name goes in the **path**, spaces and all (`/LicenceListJSON/800 MHz Band`) —
URL-encode it. Returns `LICENCE_NO`, `CLIENT_NO` and `SHIP_NAME` per licence. Note this
response omits `TOTAL_RESULT`, so use `LIC_COUNT` from step 1 to decide whether you need to
page with `strOffset`.

## Step 3 — get the full record

For a 400 MHz band licence, the widest record in the API:

```
GET /400MHZSearchJSON/{licenceOrClientNumber}/{searchTarget}?strLimit=100
```

`searchTarget` is literally `Licence No.` or `Client No.` — with the trailing full stop, URL
encoded. The response joins licence, frequency (`ASSIGN_FREQ`, in **MHz** here), emission,
bandwidth, EIRP, transmitter power, site, coverage, access area and the licensee's full
contact block, plus the band-conversion flags (`NARROW_1`, `LOWPOWER_1`, `RELOCATE_*`,
`TRANSITION_2`, `LETTER_REQUIRED`) that track the 400 MHz band replanning programme.

For any other category, fall back to:

```
GET /LicenceSearchJSON?searchText={LICENCE_NO}&searchField=LICENCE_NO
```

## Rules

- The envelope key is not the entity name. `CategoryListJSON` and `LicenceListJSON` both
  return `{"Category": [...]}` with **different row shapes**, and `400MHZSearchJSON` returns
  `{"Client No": [...]}`. Bind by operation, not by key.
- Nulls are explicit, so every documented field is present on every row.
- `STATUS` on the 400 MHz record is a raw code; `STATUS_TEXT` is the human string. Report
  `STATUS_TEXT`.
- Two operations named in ACMA's guide — `SpectrumLicenceImageSearch` and
  `ExtractSpectrumLicensingPDF` — are marked Deprecated there and are gone from the live
  WSDL. Do not attempt them.
