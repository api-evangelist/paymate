---
name: paymate-onboard-a-business
description: Onboard a business onto the PayMate global platform (KYB), submit its proof documents and shareholder records, and set the bank account its collections settle into.
api: PayMate Global Partner API
generated: '2026-08-26'
method: generated
source: openapi/paymate-global-partner-api-openapi.yml
operations:
- BusinessBoarding
- Buyersupplierkyc
- ShareholderDetails
- SetCollectionAccount
- SetTransactionCharges
- ManageBusiness
- ModifyBusiness
- TermsCondition
---

# Onboard a business on PayMate

Every other PayMate flow depends on a boarded business, so this runs first. All calls are `POST`, all
bodies are JSON, and the credential goes in a request header issued by PayMate after commercial onboarding
(see `authentication/paymate-authentication.yml` — PayMate does not publish the header name, so confirm it
with partner support before you build).

Pick the regional host that matches the business: `https://api.paymate.sg`, `https://api.paymate.my`,
`https://api.paymate.ae`, `https://api.paymate.om`, `https://api.dunomo.au`, `https://api.dunomo.co.za`.

## Steps

1. **Mint a RequestID.** Every operation requires a unique partner-generated `RequestID` and echoes it back.
   Store it before you call — it is the only trace identifier PayMate gives you, and there is no
   server-generated correlation id or request-id response header.
2. **`BusinessBoarding`** — `POST /v1/businessboarding`. Send the `BusinessInfo` object (registration number,
   business name, contact person, address, email, mobile) and, optionally, the `TransactionCharges` object.
   Use your own `BusinessCode` as the business's reference on your system; PayMate returns its own identifier.
3. **`Buyersupplierkyc`** — `POST /v1/processbuyersupplierkyc`. Upload the `BusinessProof`, `AddressProof` and
   `BankProof` documents using the document-type enum the spec declares. Expect rejections from the
   registration family of codes: `257` document already submitted, `262`/`263`/`264` empty or invalid bank
   proof, `296` KYC details already in use.
4. **`ShareholderDetails`** — `POST /v1/processShareholderdetails`. Submit the beneficial-owner records.
5. **`SetCollectionAccount`** — `POST /v1/BusinessCollectionAccount`. Send `BankAccountDetails` (account
   number, and the BIC/SWIFT or IBAN where the region uses them; Australia also carries BSB). `330`/`331`
   mean the account is already registered.
6. **`SetTransactionCharges`** — `POST /v1/SetTransctionCharges` if you did not send charges at boarding, or
   need to change them.
7. **Verify with `ManageBusiness`** — `POST /v1/managebusiness` and confirm the record reads back the way you
   sent it. Use `ModifyBusiness` (`POST /v1/modifybusiness`) to correct it.
8. **`TermsCondition`** — `POST /v1/GetTermsNCondition` returns the terms the business must accept.

## Rules

- **Read `StatusCode`, not the HTTP status.** Every call returns HTTP 200 with
  `{RequestID, StatusCode, Description, DetailedSummary}`. `"000"` is success. HTTP 401 means your credential
  headers were missing (`StatusCode` `106`). The full 508-code registry is in `errors/paymate-error-codes.yml`.
- **`DeleteBusiness` is irreversible.** There is no restore operation and no published soft-delete window.
  Treat it as human-approval-only.
- **Uniqueness errors are the common failure.** `252` business code, `254` business name, `255` email,
  `256` mobile, `327`/`328` company email/mobile, `329` reference code. Resolve them by reading back with
  `ManageBusiness` before retrying with new values — retrying the same payload will fail the same way.
