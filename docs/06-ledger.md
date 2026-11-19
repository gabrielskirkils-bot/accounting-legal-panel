# 06: General ledger, posting engine, sub-registers, period close, year end

The app **is** the bookkeeping system. Postgres holds the journal. A Moneo export is optional (for a bookkeeper or for archive).

## Chart of accounts

- Phase 0 seeds a standard Latvian double-entry chart **per entity**, with a Bourzma (association) variant and a GES (SIA) variant. Build it from the account classes used in Latvian practice, and **look up and cite** the structure; don't recall it. Gabriels approves it under D-POL-2.
- If an existing chart is available (e.g. from Moneo or last year's report), import it instead and map the categories to it.
- Each account has:
  - class;
  - type (asset, liability, equity, income, expense);
  - normal balance;
  - a default VAT code;
  - the **annual-report line** it rolls up to: the bilance or ieņēmumu/izdevumu line of MK 439 (Bourzma), or the balance-sheet or P&L line (GES).

  That mapping makes the annual report a query, not manual work.
- Category folder → default account mapping: Telpu noma → rent; Grāmatvedības pakalpojumi → professional services; Video pakalpojumi / Pasākuma izdevumi → services or event costs; materials → materials; Mārketinga izdevumi → advertising; 6110 → revenue; 7170 → transport; 7550 → services and subscriptions; 7720 → small equipment, or a fixed asset after the D-ASSET-1 check.

## Posting engine

A deterministic template per transaction `type` and `vat_regime`. The same input always gives the same entry. Every entry links to its transaction record and the rule used.

| Event | Debit | Credit |
|---|---|---|
| Purchase invoice | expense / asset (+ deductible input VAT) | supplier payable |
| Non-deductible VAT share | added to the expense / asset | |
| Payment of a purchase | supplier payable | bank |
| Card receipt | expense + input VAT | bank (via the card's account) |
| Paid privately by a person | expense + input VAT | payable to that person |
| Sale invoice | customer receivable | revenue + output VAT |
| Customer payment | bank | customer receivable |
| EU reverse-charge service | expense + input VAT (deductible share) | supplier payable + output VAT (self-assessed) |
| Grant received (D-POL-5) | bank | deferred grant income |
| Grant recognised (monthly, with the eligible costs) | deferred grant income | grant income |
| Donation | bank | donation income (also the ziedojumu un dāvinājumu pārskats register) |
| Loan from a person | bank | loan payable (short or long term) |
| Loan repayment | loan payable | bank |
| Cash withdrawal / cash spending | cash ← bank / expense ← cash | |
| Private-person payment with tax (D-PRV-1) | expense | payable to the person (net) + tax payable (IIN, VSAOI per `rules`) |
| Tax payment to VID | tax payable | bank |
| Bank fee | bank fees | bank |
| Depreciation (monthly) | depreciation expense | accumulated depreciation |
| Accrual (service period ≠ posting month) | expense | accrued liabilities (reversed next month) |
| FX difference on settlement | FX loss / gain | |
| VAT period close | output VAT − input VAT → VAT payable / receivable account | |

- **Corrections never edit a posted entry.** They post a reversal and a new entry, both linked.
- **Each closed period is locked.** A late document goes into the current open period, with a reference to the original period (and a VAT correction if needed, see `07`).

## Sub-registers (all derived from the journal and the transactions; always tie to the control accounts)

| Register | Contents | Must equal |
|---|---|---|
| Payables (kreditori) | open items per supplier, ageing, debt-collector claims kept separate | the payable control account |
| Receivables (debitori) | open items per customer, ageing | the receivable control account |
| Bank | lines per account; balance per statement | the bank account balance |
| Fixed assets | card per asset: cost, date, life, depreciation, net book value | the asset accounts |
| Low-value items | list (D-POL-4) | |
| Loans | per lender: agreement, movements, short/long-term split | the loan accounts |
| Cash book (kases grāmata) | orders, balance | the cash account |
| Grants | per grant: received, recognised, spent per budget line, remaining | deferred grant income |
| Donations and gifts | donor, date, amount, purpose, use (Bourzma) | donation income |
| Persons | amounts owed to or by board members and staff | the person accounts |

## Month close (D-CLOSE-1)

1. All bank lines for the month are imported and their balances tie out.
2. No `awaiting_decision` records older than the booking deadline.
3. Depreciation, accruals, grant recognition and FX are posted automatically.
4. Control checks (`08`) are green, or each warning is accepted with a reason.
5. Gabriels closes the month, and the period locks.

## Year end (D-YEAR-1, D-YEAR-2)

- **Inventory count (D-POL-10).** The app prepares the inventory lists (cash, bank confirmations, receivables and payables confirmations, fixed assets), and **drafts reconciliation acts in Latvian** for the main counterparties for Gabriels to send.
- **Proposals:**
  - doubtful receivables;
  - accrued costs and income;
  - grant deferral;
  - short/long-term reclassification;
  - provisions (e.g. claimed collector fees, if decided).
- **Close the year:** income and expense accounts go to the result (Bourzma: surplus/deficit; GES: retained earnings).
- **Annual-report draft from the account-to-line mapping:**
  - Bourzma, MK 439 double-entry: bilance, ieņēmumu un izdevumu pārskats, ziedojumu un dāvinājumu pārskats, paskaidrojums, vadības ziņojums;
  - GES: per the Gada pārskatu un konsolidēto gada pārskatu likums and its size category.

  Comparative figures come from last year's report. Where VID publishes an XSD for the annual report, build the XML; otherwise produce a filled table for entry in EDS.
- **Next year's opening balances** are the closing balances, carried forward automatically.
