# 05: Decision catalog. Exactly what Gabriels decides.

This is the complete list. Anything not listed is decided automatically from `rules`, `policies` and counterparty history, and every automatic choice is still visible and reversible in the record.

**Every decision card shows:**
- the question;
- 2–5 options;
- a **recommended option**, with a one-line reason and the legal source;
- the money affected;
- a preview of the result.

The **scope** options are *just this one / always for this counterparty / always (policy)*. Answers with a wider scope become rules, so the same question isn't asked twice.

## Everyday (per transaction)

| Code | Question | Options | Recommended default |
|---|---|---|---|
| **D-ENT-1** | Which entity is this? | Bourzma / Green Echo / MNB (not booked) / Personal | The suggestion from `04` step 8. One click; batch "confirm all ≥ 95 %" |
| D-ENT-2 | The buyer is a private person (e.g. Patriks). What is it? | Personal (not booked) / Company expense paid privately (payable to the person) / Wrong buyer, ask the supplier to reissue | Reissue if over 50 EUR, otherwise personal |
| D-ENT-3 | Money moved between Bourzma, GES or MNB. What is it? | Loan / Payment of an internal invoice / Recharge of costs / Mistake to refund | Ask for the internal invoice |
| D-DOC-1 | Two versions of the same document. Which one is valid? | Version A / Version B | The one equal to the bank payment |
| D-DOC-2 | Payment with no document after 14 days | Draft a request to the supplier / Mark as personal / Book on a substitute document (izdevumu attaisnojuma akts, per `rules`) | Draft a request |
| D-INC-1 | Money received without an issued invoice | Issue an invoice now (draft created) / Grant / Donation / Refund / Other income | Issue an invoice if from a company, grant if from a funder |
| D-DUP-1 | Possible double payment | Ask for a refund (draft) / Offset against the next invoice / It's a second invoice (attach it) | Ask for a refund |
| D-DEBT-1 | Debt-collector or creditor claim | Pay principal only / Pay all / Dispute (draft letter) / Negotiate | Pay principal, dispute fees above the legal cap (per `rules`) |
| D-VAT-1 | VAT treatment not clear (mixed use, unusual regime) | Full / None / Proportional / Different regime | From the rules; only asked when the rules can't decide |
| D-ASSET-1 | Above the capitalisation threshold: fixed asset? | Fixed asset (useful life …) / Expense / Low-value item | Fixed asset if the useful life is over 1 year |
| D-PRV-1 | Payment to a private person (author fee, prize, honorarium) | Income type and tax treatment options from `rules` | From the contract type in the payment details |
| D-PERS-1 | Card spending marked personal | Person repays (receivable) / Offset against the person's loan / Treat as a benefit (taxable, per `rules`) | Person repays |
| D-CASH-1 | Cash spending without a receipt | Attach a receipt / Expense order (izdevumu orderis) with purpose / Personal | Attach a receipt |
| D-GRANT-1 | Draft grant found in the bank data | Confirm (name, period, budget lines, eligible costs) / Not a grant | Confirm with the funder's agreement attached |
| D-PRJ-1 | Draft project found | Confirm (customer, invoicing entity) / Ignore | Confirm |
| D-ROUTE-1 | Planning: which entity should buy this? | Rule result: grant → actual use and VAT → project | The rule result; see the routing section |

## Periodic

| Code | When | Question | Default |
|---|---|---|---|
| D-CLOSE-1 | Month end (+10 days) | Close the month? The checklist from `08` must be green, or each warning accepted | Close when green |
| D-VAT-2 | Each VAT period | Approve the VAT return and download the export | Approve when green |
| D-VAT-3 | Once a year per entity (Bourzma) | Input-VAT deduction rate for mixed use: the app calculates a proposal from the year's taxable vs total income, per the formula in `rules` | Accept the calculation |
| D-PRV-2 | Monthly / by 1 February | Approve the paziņojums (notice of payments to private persons) export | Approve |
| D-YEAR-1 | Year end | Year-end adjustments: doubtful receivables, accruals, provisions, inventory count result, grant deferral, loan reclassification (short/long term) | The app's proposals, one card each |
| D-YEAR-2 | After the year-end close | Approve the annual-report draft and the management report text | Approve the draft |

## Legal & Authorities (detail in `12` section 4)

| Code | Question | Default |
|---|---|---|
| D-LEG-1 | New legal item: entity (incl. MNB), matter, type | Suggestion from addressee + reference no. |
| D-LEG-2 | Approve assessment, option and plan | Recommended option, with reason and law |
| D-LEG-3 | Approve the final letter (Gabriels sends it) | Approve |
| D-LEG-4 | Ask for a deadline extension (draft ready) | Yes, if the answer pack is incomplete 5 working days before |
| D-LEG-5 | Bring in a lawyer | Yes, for any escalation trigger |
| D-LEG-6 | Close the matter | Close when the outcome is recorded and no deadline is open |
| D-CON-1 | Confirm a new contract's parties, term, money, obligations | The extraction |
| D-CON-2 | Notice deadline coming: renew / renegotiate / terminate | Ask 30 days ahead |

## One-time policies (Phase 0; together they make the accounting organization documents)

| Code | Policy | Recommended start value (verify against `rules`) |
|---|---|---|
| D-POL-1 | Who keeps the books (legal gate 1 in `01`) | Per entity: self-kept board member / bookkeeper under agreement |
| D-POL-2 | Chart of accounts | The app's standard chart for Latvian double entry (Bourzma: association variant; GES: SIA variant) |
| D-POL-3 | Capitalisation threshold and depreciation method | Straight-line; threshold from `rules` |
| D-POL-4 | Low-value items | Expense on purchase, listed in the low-value register |
| D-POL-5 | Grant accounting | Grant received → deferred income; recognised as income in step with eligible costs |
| D-POL-6 | Foreign-currency rate source | The ECB/Bank of Latvia rate on the transaction date (verify in `rules`) |
| D-POL-7 | Cash rules | Cash book with orders; maximum cash balance and per-payment limits per `rules` |
| D-POL-8 | Representation and gifts | Purpose and participants required; VAT and tax treatment per `rules` |
| D-POL-9 | Personal use of company cards | Repay within 30 days |
| D-POL-10 | Inventory count (inventarizācija) | Annually at 31.12: cash, bank, receivables, payables, fixed assets |

## Routing: which entity should buy (D-ROUTE-1)

Apply in order, and show the rule that decided:

1. **Grant.** An active grant covers the cost → the grant's entity. Show the budget line and the remaining amount.
2. **Actual use, then VAT.** Input VAT is deductible only by the entity that uses the purchase for its own taxable supplies. So first establish whose activity uses it. If both entities could genuinely use it, prefer the one with the higher deduction rate. Never route a cost to one entity when the other one consumes it.
3. **Project.** A project paid for by one customer → the entity that invoices that customer.
4. Otherwise, ask.
