# ACMA (acma)

The Australian Communications and Media Authority (ACMA) is the Commonwealth regulator for telecommunications, radiocommunications, broadcasting and online content in Australia. It issues carrier licences and apparatus and spectrum licences, maintains the Register of Radiocommunications Licences, administers the Australian telephone numbering plan through the Numbering System, operates the Do Not Call Register, and polices the Telecommunications Consumer Protections and scam-call rules that bind Telstra, Optus, TPG and every other Australian carrier.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/acma/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/acma/refs/heads/main/apis.yml)

## Tags

- Telecommunications
- Australia
- Regulator
- Spectrum
- Broadcasting
- Numbering
- Do Not Call Register
- Radiocommunications
- Licensing
- Open Data
- Government
- SOAP

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## APIs

### ACMA Spectrum Licensing API

A completely public, entirely anonymous REST + SOAP API over the Register of Radiocommunications Licences. No API key, no account, no registration, no `Authorization` header — every one of its 22 operations answers an anonymous request. Eleven web methods are each published in an XML and a JSON projection: licence search, client (licensee) search, site search, site-by-location proximity search, device registration search, assignment search by postcode/frequency/date range, antenna register search, access area lookup, licence category list, licences-in-category list, and the 400 MHz band spectrum licence register search.

A single query returns at most **2,000 records** — a deliberate cap "to discourage people from downloading the whole dataset through the API" — and callers page past it with `offset`. For whole-dataset work ACMA publishes a same-day complete extract as a zip download instead.

- **Human URL:** [https://www.acma.gov.au/radiocomms-licence-data](https://www.acma.gov.au/radiocomms-licence-data)
- **Base URL:** `https://api.acma.gov.au/SpectrumLicensingAPIOuterService/OuterService.svc`
- **Live WSDL:** [`OuterService.svc?wsdl`](https://api.acma.gov.au/SpectrumLicensingAPIOuterService/OuterService.svc?wsdl) — 22 operations, harvested verbatim to [`wsdl/`](wsdl/)
- **Documentation:** [Spectrum Licensing API guide (DOCX, v1.0, 17 June 2016)](https://www.acma.gov.au/sites/default/files/2019-11/Spectrum%20licensing%20API.docx)

#### Properties

- [OpenAPI 3.1 (reconstructed)](openapi/acma-spectrum-licensing-openapi.yml) — 22 operations, 22 schemas
- [WSDL (verbatim)](wsdl/acma-spectrum-licensing.wsdl)
- [Overlay](overlays/acma-spectrum-licensing-overlay.yaml)
- [Live response examples](examples/)
- [Data model](data-model/acma-data-model.yml) · [Errors](errors/acma-problem-types.yml) · [MCP](mcp/acma-mcp.yml) · [Tool crosswalk](mcp/acma-tool-crosswalk.yml) · [Skills](skills/_index.yml) · [Agentic access](agentic-access/acma-agentic-access.yml) · [Arazzo](arazzo/_index.yml)

### Do Not Call Register Real Time Access (RTA) Washing Service

The Do Not Call Register is operated by the ACMA under the Do Not Call Register Act 2006. Telemarketers and fax marketers must check ("wash") their contact lists against the register before calling. Four submission channels exist, two of them programmatic — "Automated Washing Service (AWS/SFTP)" and "Real Time Access (RTA/SOAP)" — alongside manual Quick Wash keying and website upload.

The RTA channel is a ColdFusion SOAP service with three operations — `GetAccountBalance`, `WashNumbers`, `GetWashResult` — whose **WSDL is publicly retrievable**. Calling it requires an industry account and a paid subscription of type D or above; credentials are an account ID and passphrase carried in the SOAP body, and TLS 1.2+ is mandatory. Up to 200 numbers per call is the published optimum, 500 the maximum, and fewer than 5 million per month. Results are `Y` (on the register), `N` (not on it) or `I` (invalid).

- **Human URL:** [https://www.donotcall.gov.au/industry/washing-process-overview/washing-steps](https://www.donotcall.gov.au/industry/washing-process-overview/washing-steps)
- **Base URL:** `https://www.donotcall.gov.au/dncrtelem/rtw/washing.cfc`
- **Live WSDL:** [`washing.cfc?wsdl`](https://www.donotcall.gov.au/dncrtelem/rtw/washing.cfc?wsdl) — harvested to [`wsdl/`](wsdl/)

#### Properties

- [WSDL (verbatim)](wsdl/acma-dncr-realtime-washing.wsdl)
- [System codes](errors/acma-dncr-system-codes.yml) · [Subscriptions and fees](plans/acma-plans.yml) · [Skill](skills/acma-wash-numbers-do-not-call-register.md)

## About the OpenAPI in this repo

ACMA publishes **no** OpenAPI. The document in [`openapi/`](openapi/) is an API Evangelist reconstruction: paths, parameters, enumerated values and operation semantics are taken verbatim from ACMA's published guide and the live WSDL, and response schemas are derived from live responses captured on 2026-07-25. **Every operation carries an `x-evidence` block** naming its documentation source and the live URL that was probed, all of which returned HTTP 200.

## What is not here

- **No OpenAPI, Swagger or AsyncAPI published by ACMA.** Probed `/openapi.json`, `/openapi.yaml`, `/swagger.json`, `/v1/openapi.json`, `/swagger/v1/swagger.json`, `/api-docs`, `/docs` and `/redoc` across `api.acma.gov.au`, `developer.acma.gov.au`, `www.donotcall.gov.au` and `thenumberingsystem.com.au` — all 404 or SPA shell.
- **Developer portal.** [https://developer.acma.gov.au/](https://developer.acma.gov.au/) is a real Microsoft Azure API Management developer portal — "Sign in to the ACMA's API portal" — but every anonymous request, including `/apis` and `/products`, returns the same sign-in shell, and the portal's own management API returns HTTP 401. It is a login wall, and it is **not** where the Spectrum Licensing API lives.
- **API gateway root.** [https://api.acma.gov.au/](https://api.acma.gov.au/) answers HTTP 403 on `/` and 404 on short paths. The live service sits at the full WCF path above.
- **Numbering System.** [https://www.thenumberingsystem.com.au/](https://www.thenumberingsystem.com.au/) is an authenticated web application for licensed carriage service providers whose internal JSON backend returns HTTP 401 anonymously. Not a documented public API.
- **Open data.** ACMA's registers are also published on [data.gov.au](https://data.gov.au/data/organization/australiancommunicationsandmediaauthority) as 8 datasets of PDF and file downloads. None are DataStore-active, so there is no query API over them.
- **No `/.well-known/` documents on any host** — including no `security.txt`, despite a real [vulnerability disclosure policy](https://www.acma.gov.au/vulnerability-disclosure-policy) and a `responsible.disclosure@acma.gov.au` mailbox.
- **CAMARA / GSMA Open Gateway.** No reference found anywhere — not even a press release. ACMA is a regulator, not an operator, and CAMARA plus Open Gateway are operator-side commitments. Its bearing on Australia's network-API surface is regulatory: it administers the numbering plan, the carrier licences, and the anti-scam and identity-verification rules operators must satisfy.
- **TM Forum, 3GPP NEF/SCEF, webhooks, AsyncAPI, GraphQL, gRPC, Postman, SDKs, CLI, status page, dated changelog, GitHub organisation.** None found. Generate a client from the WSDL instead.

Full reviewer notes, probe results and HTTP statuses are in [`review.yml`](review.yml).

## Links

- [Website](https://www.acma.gov.au/)
- [Radiocomms licence data (documents the API)](https://www.acma.gov.au/radiocomms-licence-data)
- [Register of Radiocommunications Licences](https://www.acma.gov.au/register-radiocommunication-licences-rrl)
- [Offline RRL tool](https://web.acma.gov.au/offline-rrl/)
- [Developer portal (sign-in required)](https://developer.acma.gov.au/)
- [Do Not Call Register](https://www.donotcall.gov.au/)
- [The Numbering System](https://www.thenumberingsystem.com.au/)
- [mySwitch](https://myswitch.digitalready.gov.au/)
- [Vulnerability disclosure policy](https://www.acma.gov.au/vulnerability-disclosure-policy)
- [ACMA data on data.gov.au](https://data.gov.au/data/organization/australiancommunicationsandmediaauthority)
- [LinkedIn](https://www.linkedin.com/company/australian-communications-and-media-authority)
