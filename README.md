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
- Open Data
- Government

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## APIs

### Do Not Call Register Washing Service

The Do Not Call Register is operated by the ACMA under the Do Not Call Register Act 2006. Telemarketers and fax marketers must check ("wash") their contact lists against the register before calling. ACMA documents four submission channels, two of which are programmatic — "Automated Washing Service (AWS/SFTP)" and "Real Time Access (RTA/SOAP)" — alongside manual Quick Wash keying and website file upload. Access requires an industry account with a paid washing subscription; no WSDL, OpenAPI definition, endpoint host, or credential documentation is published anonymously.

- **Human URL:** [https://www.donotcall.gov.au/industry/washing-process-overview/washing-steps](https://www.donotcall.gov.au/industry/washing-process-overview/washing-steps)

#### Tags

- Do Not Call Register
- Telemarketing
- Compliance
- SOAP
- SFTP

#### Properties

- [Documentation](https://www.donotcall.gov.au/industry/washing-process-overview/washing-steps)
- [Documentation](https://www.donotcall.gov.au/industry/washing-process-overview)
- [Documentation](https://www.donotcall.gov.au/industry/industry-overview/)

## What is not here

This repo carries no `openapi/` directory because no machine-readable specification of any kind was found.

- **Developer portal.** [https://developer.acma.gov.au/](https://developer.acma.gov.au/) is a real Microsoft Azure API Management developer portal — its page reads "Sign in to the ACMA's API portal" — but every anonymous request, including `/apis` and `/products`, returns the same sign-in shell, and the portal's own management API returns HTTP 401. It is a login wall, not a self-serve portal.
- **API gateway.** [https://api.acma.gov.au/](https://api.acma.gov.au/) answers HTTP 403 on root and 404 on every probed path. No public surface, no documentation.
- **Numbering System.** [https://www.thenumberingsystem.com.au/](https://www.thenumberingsystem.com.au/) is an authenticated web application for licensed carriage service providers with an internal JSON backend that returns HTTP 401 anonymously. Not a documented public API.
- **Open data.** ACMA's registers and licence data are published on [data.gov.au](https://data.gov.au/data/organization/australiancommunicationsandmediaauthority) as 8 datasets of PDF and file downloads. None of the resources are DataStore-active, so there is no query API over them.
- **CAMARA / GSMA Open Gateway.** No reference found anywhere — not even a press release. ACMA is a regulator, not an operator, and CAMARA plus Open Gateway are operator-side commitments. Its bearing on Australia's network-API surface is regulatory: it administers the numbering plan, the carrier licences, and the anti-scam and identity-verification rules operators must satisfy.
- **TM Forum, 3GPP NEF/SCEF, webhooks, AsyncAPI, GraphQL, gRPC, Postman, SDKs, GitHub organisation.** None found.

Full reviewer notes, probe results and HTTP statuses are in [`review.yml`](review.yml).

## Links

- [Website](https://www.acma.gov.au/)
- [Developer portal (sign-in required)](https://developer.acma.gov.au/)
- [Do Not Call Register](https://www.donotcall.gov.au/)
- [The Numbering System](https://www.thenumberingsystem.com.au/)
- [ACMA data on data.gov.au](https://data.gov.au/data/organization/australiancommunicationsandmediaauthority)
- [LinkedIn](https://www.linkedin.com/company/australian-communications-and-media-authority)
