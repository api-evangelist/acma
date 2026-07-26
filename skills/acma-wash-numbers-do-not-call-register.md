---
name: Wash a contact list against the Do Not Call Register
description: >-
  Check Australian phone numbers against the ACMA Do Not Call Register over the Real Time
  Access SOAP channel — balance, submit, recover from timeout — including the metering and
  legal rules that make this the one ACMA call an agent can get expensively wrong.
api: wsdl/acma-dncr-realtime-washing.wsdl
operations:
  - GetAccountBalance
  - WashNumbers
  - GetWashResult
generated: '2026-07-25'
method: generated
---

# Wash a contact list against the Do Not Call Register

Unlike the rest of ACMA's public surface, this operation **costs money and carries legal
weight**. Under the Do Not Call Register Act 2006 a telemarketer must wash a list before
calling it, and every number submitted — valid or not — consumes a wash credit.

## Preconditions

- An industry account (administration, or a wash-only sub-account) at
  <https://www.donotcall.gov.au/industry/create-account>.
- A paid subscription of **type D or above** — types A, B and C cannot use the programmatic
  channels at all.
- **TLS 1.2 or above**, forced in the client. ACMA rejects older connections.
- WSDL: `https://www.donotcall.gov.au/dncrtelem/rtw/washing.cfc?wsdl` (public — generate the
  client from it; there is no SDK). Endpoint:
  `https://www.donotcall.gov.au/dncrtelem/rtw/washing.cfc`.

Credentials are SOAP body parameters, not HTTP auth: `TelemarketerId` (the Account ID) and
`TelemarketerPassword`. Add `WashOnlyUserId` only when washing under a wash-only sub-account.
The passphrase must match the one used when MFA was configured on the account; MFA itself is
not applied to this channel.

## Step 1 — check the balance first

Call `GetAccountBalance` before every batch. It returns the available wash credits. This is
the cheap call; do not discover an empty balance halfway through a list.

System codes: `0` success, `1` missing/non-numeric Telemarketer ID or password, `3` account
does not exist.

## Step 2 — validate the numbers yourself

ACMA does **not** warn about invalid entries and does **not** offer a chance to abort —
invalid entries are processed and **still charged**. Validation is explicitly the caller's
responsibility.

Permitted characters in a submitted number: digits, brackets `(` `)`, hyphens `-` and spaces,
e.g. `(03) 9999-8888`.

## Step 3 — submit

Call `WashNumbers` with `TelemarketerId`, `TelemarketerPassword`, a `ClientReferenceId` and
the numbers.

- **`ClientReferenceId` is the idempotency handle.** It must be unique within your
  organisation, and it is the only way to retrieve the result of a request that timed out
  without paying to wash the list a second time. Generate and persist it *before* the call.
- **Batch size: up to 200 numbers per call** is ACMA's stated optimum; **500 is the maximum**
  and will time out under load. Larger lists belong on the AWS/SFTP channel.
- **Stay under 5 million numbers per month** on this channel.

Per-number results: `Y` = the number **is** on the register (do not call it), `N` = not on
the register, `I` = invalid phone number.

System codes: `0` success, `1` missing/non-numeric credentials or numbers, `3` account
missing/inactive/suspended, `4` subscription missing or expired, `5` insufficient balance.

## Step 4 — recover from a timeout, do not resubmit

Timeouts are expected, not exceptional. ACMA's published recovery procedure:

1. Catch the timeout. **Do not call `GetWashResult` immediately.**
2. Wait **at least 30 seconds**, then call `GetWashResult` with the same `ClientReferenceId`.
3. If that times out or returns "Client Reference ID does not exist", wait **another 30
   seconds** and try again.
4. If it fails a third time, wait **at least 30 minutes**, or give up.
5. If timeouts are frequent, do not just retry harder — move the wash to off-peak hours or to
   the SFTP channel. Spread load "across seconds (not hours or daily)".

`GetWashResult` system codes: `0` success, `2` Client Reference ID does not exist, `3`
account does not exist, `4` washing started but not yet finished. Code `4` means keep
waiting; it is not a failure.

## Legal rules an agent must not optimise away

- A washed list can generally be relied on for **30 days**. After that it must be washed
  again before calling.
- Never resubmit a batch to "make sure" — it double-charges and does not change the answer.
  Use `GetWashResult`.
- A `Y` result is a prohibition, not a preference.

## Batch alternative

For large lists, use the Automated Washing Service over SFTP at `sftp.donotcall.gov.au`
(upload / download / archive folders, SSH-key authentication, IP-restricted). Turnaround is
usually under a minute. See <https://www.donotcall.gov.au/industry/washing-process-overview/sftp>.
