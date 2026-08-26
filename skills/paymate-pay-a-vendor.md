---
name: paymate-pay-a-vendor
description: Pay a supplier from a business's enrolled commercial credit card on PayMate, then track the payment to settlement.
api: PayMate Global Partner API
generated: '2026-08-26'
method: generated
source: openapi/paymate-global-partner-api-openapi.yml
operations:
- ContactBoarding
- ManageContacts
- AddCard
- ManageCards
- PaymentType
- MakePayments
- GetPaymentStatus
- PaymentReport
---

# Pay a vendor with PayMate

This is the money-movement path. **Nothing here can be undone through the API** — PayMate publishes no
refund, void, cancel or reverse operation. Get human approval before step 5.

## Steps

1. **Board the supplier as a contact** — `ContactBoarding`, `POST /v1/contactboarding`. Send
   `ContactInformation` plus `BankAccountDetails`. Confirm with `ManageContacts`
   (`POST /v1/managecontact`), which returns the contact with its bank and KYC detail.
2. **Enrol the funding card** — `AddCard`, `POST /v1/AddCard`. This takes a raw card number, expiry and CVV,
   and PayMate restricts it: *"Only PCI certified customer can use this api."* If you are not PCI certified,
   the business must add the card through PayMate's own dashboard instead.
3. **Complete the 3-D Secure challenge.** `AddCard` can return `StatusCode` "Token creation in process" with a
   `FormData` field containing an HTML 3-D Secure form. Render it to the cardholder; the token is not usable
   until the challenge completes. Confirm the card is live with `ManageCards` (`POST /v1/managecard`).
4. **Check the available payment types** — `PaymentType`, `POST /v1/Paymenttype`.
5. **Initiate the payment** — `MakePayments`, `POST /v1/MakePayment`. Send `transactionDetails`, the `invoice`
   array and any `debitNote` entries. **Set your own `OrderID`** and persist it before the call.
   In Australia there is an additional `VendorPayments` operation (`POST /v1/VendorPayment`).
6. **Poll status** — `GetPaymentStatus`, `POST /v1/getpaymentstatus`, keyed on `OrderID` or the
   `PayMateRequestNo` PayMate returned. The response carries `TransactionDetails`,
   `RemitterAccountDetails`, `SettlementDetails` and `RefundDetails`.
7. **Reconcile** — `PaymentReport`, `POST /v1/PaymentHistory`, over a `FromDate`/`ToDate` window. There is no
   pagination; narrow the window instead. `235` means the range is invalid.

## Rules

- **Retry only with the same `OrderID`.** A repeat submission of a processed order is rejected with
  `413 OrderId already processed`, which is the only thing standing between a retry and a double payment.
  Never mint a fresh `OrderID` to "make the retry go through".
- **`RefundDetails` is not a refund you can ask for.** It is populated when PayMate's own settlement fails —
  `538` debit bank failure, `539` credit bank failure — and returns money automatically. There is no
  caller-invokable reversal and no published window.
- **Watch the limit codes.** `140`/`417` amount limit exceeded, `542` transaction limit exceeded. Monthly
  transaction caps are commercial (1,000/month on Basic API, 3,000/month on Enterprise API) and are billed
  as overage rather than signalled in a header — see `rate-limits/paymate-rate-limits.yml`.
- **Payout timing is not an SLA.** Card, debit-card and net-banking transactions credit the payee within
  48 hours; NEFT/RTGS/IMPS run during banking hours only.
