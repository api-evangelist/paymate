---
name: paymate-collect-from-a-customer
description: Raise a PayMate collection request against a customer, then track it through to settlement and reconcile it.
api: PayMate Global Partner API
generated: '2026-08-26'
method: generated
source: openapi/paymate-global-partner-api-openapi.yml
operations:
- ContactBoarding
- ManageContacts
- SetCollectionAccount
- CollectPayment
- CollectionStatus
- CollectionReport
---

# Collect a payment with PayMate

## Steps

1. **Make sure the business has a collection account** — `SetCollectionAccount`,
   `POST /v1/BusinessCollectionAccount`. Collections settle into this account; without it there is nowhere
   for the money to land.
2. **Board the customer as a contact** — `ContactBoarding`, `POST /v1/contactboarding`, then verify with
   `ManageContacts` (`POST /v1/managecontact`).
3. **Raise the request** — `CollectPayment`, `POST /v1/collectpayments`. Send `CollectionDetails`, the
   `Invoice` array, and `SplitMDR` when the merchant discount rate is to be split with the payer. Set your own
   `OrderID`. PayMate delivers the request to the customer as a payment link (the platform sends these by
   email, WhatsApp or SMS).
4. **Track it** — `CollectionStatus`, `POST /v1/GetCollectStatus`. The response carries
   `TransactionDetails`, `Invoice`, `CollectionSummary`, `SettlementDetails` and `RefundDetails`.
5. **Reconcile** — `CollectionReport`, `POST /v1/CollectHistory`, over a `FromDate`/`ToDate` window.

## Rules

- **The customer pays, not you.** A collection request is an instruction to a third party; you control when
  it is raised, not when it is paid.
- **There is no cancel operation.** Once `CollectPayment` succeeds, PayMate publishes no way to withdraw the
  request through the API. Confirm the amount and the payer before you call.
- **Settlement is T+2 and free**; card collections carry an MDR (2.5% on the API plans, 2.25% on Enterprise
  Custom, all plus GST) which `SplitMDR` decides who pays. See `plans/paymate-plans-pricing.yml`.
- **Read `StatusCode`.** HTTP 200 is returned on business failure too. Codes are in
  `errors/paymate-error-codes.yml`.
