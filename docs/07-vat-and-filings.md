# 07: VAT (PVN) and filings

The app prepares every filing as a file Gabriels uploads to VID EDS himself. **It never logs in to EDS.** Every rate, threshold, form and deadline lives in `rules`, with its source URL (likumi.lv / vid.gov.lv) and a `valid_from` date. Never hard-code them.

## Tax periods

| Entity | PVN period | Due |
|---|---|---|
| Green Echo Studio | month | the 20th of the next month (per `rules`) |
| Bourzma | quarter | the 20th of the month after the quarter (per `rules`) |

## VAT registers (built from the transaction records, not typed)

- **Purchases:** supplier, VAT number (VIES-checked), invoice number and date, net, VAT, rate, regime, deductible share, and the PVN 1 row it feeds.
- **Sales:** customer, VAT number, invoice number and date, net, VAT, rate, regime.
- Each register must equal the VAT ledger accounts for the period (a control in `08`).

## Regimes handled

- **Domestic standard and reduced rates**, from `rules`.
- **Reverse charge.** EU services (e.g. Meta, Google, Adobe, OpenAI) and domestic reverse charge where `rules` say so. Output and input VAT are both self-assessed.
- **EU goods acquisitions.**
- **Exempt and out-of-scope.** Grants, donations and membership fees: per `rules`, these usually sit outside VAT, but they affect Bourzma's deduction rate.
- **Partial deduction (Bourzma).** During the year, the provisional rate from D-VAT-3 applies. At year end, the app recalculates the rate from actual figures and posts the adjustment as a card.
- **Non-deductible items:** representation and passenger cars, per `rules`.

## PVN declaration (D-VAT-2)

1. **Pre-check (`08`).** All bank lines for the period are matched, there are no records `awaiting_decision` in the period, all VIES checks are done, and the registers equal the ledger.
2. **Build PVN 1** with its parts:
   - **Part I:** the summary rows.
   - **Part II:** input-tax itemisation.
   - **Part III:** output-tax itemisation.

   Use the itemisation threshold from `rules` (currently 150 EUR excluding VAT per MK 40; verify). Add the EU annexes (PVN 2) when there are EU supplies.
3. **Generate the export.** Phase 3 builds the PVN data as a clean Excel file laid out like PVN 1 (one tab per part) plus the totals Gabriels or the bookkeeper types into EDS. The exact export format is **adjustable later** (decided 19.11.2026): the generator is one module with a pluggable writer, so an EDS XML writer can be added without touching the registers. If VID XSD files are uploaded in `Settings → Schemas`, the XML writer validates against them and a failing file is never offered; without them, XML is simply not offered.
4. **Show it side by side:** each PVN row, its figure, and a drill-down to the transactions behind it.
5. Gabriels approves (D-VAT-2) and downloads the export. He files it in EDS and enters the EDS submission number back in the app. The VAT period then locks.
6. **Late documents** for a locked period go into a corrective declaration (labojums) or into the current period, as `rules` allow. The app proposes which.

## Other filings

| Filing | Who | When | Source |
|---|---|---|---|
| Paziņojums par fiziskajai personai izmaksātajām summām (notice of payments to private persons) | both entities, when they pay private persons | by the 15th of the next month if tax was withheld; otherwise by 1 February (per `rules`) | D-PRV-1, D-PRV-2 |
| Darba devēja ziņojums (employer report) / VSAOI | when there are salaries (none yet) | monthly, per `rules` | disabled until the first employee |
| Gada pārskats (annual report), Bourzma | association, double-entry, MK 439 | by 31 March (verify) | `06` year end |
| Gada pārskats, GES | SIA, by size category | per the law (verify) | `06` year end |
| UIN deklarācija (corporate income tax), GES | monthly, only when there are taxable events (dividends, deemed distributions) | per `rules` | the ledger flags deemed distributions, e.g. personal expenses not repaid |
| Bourzma, public-benefit status (if any) | per `rules` | | ask in Phase 0 |

## Filing calendar

- Generated from `rules` for both entities.
- Shown in the **Taxes** screen and the **Today** strip.
- Reminders at T-7 and T-2 days by email or WhatsApp.
- Each item shows a status: `not started` → `ready to approve` → `downloaded` → `submitted (EDS no.)`.

## Retention (Grāmatvedības likums Art. 28)

| Kind | Kept |
|---|---|
| Supporting documents | at least 5 years |
| Registers and accounting organization documents | 10 years |
| Annual reports | until the entity ends |

Drive is the archive. Storage objects are never hard-deleted while their retention period is running.
