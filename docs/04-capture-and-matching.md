# 04: Capture, document processing, bank import, matching and filing

## A. Email

- One connected mailbox: **gabrielsk@mnbproduction.com** (Gmail API, read-only).
- **Auto-forwarding into it** (Gabriels switches it on; Settings shows a checklist and the date of the last mail per forwarded address, with a warning after 14 days of silence):
  - info@bourzma.com → gabrielsk@;
  - Patriks's CityBee mail (spended0@gmail.com, from info@citybee.lv / .lt) → gabrielsk@.
- **Forwarded mail is treated as the original:** take sender, date and subject from the forwarded headers or the attached .eml, and store them in `sources.original_sender` and `received_at`.
- **Triage** with the cheap model on subject, sender, snippet and attachment names. Signals: attachments (pdf, jpg, png, xls/xlsx, doc/docx, edoc), and keywords *rēķins, rekins, invoice, receipt, kvīts, čeks, pavadzīme, faktūra, priekšapmaksa, avansa, salīdzināšanas akts, atgādinājums, brīdinājums, parāds, inkasso, pretenzija, līgums*.
- **Legal mail** (authorities, courts, bailiffs, collectors, contracts) is routed to the Legal inbox per `12` section 1, in addition to the invoice queue when it also carries an invoice. EDS and e-adrese notification emails create a "download from EDS / e-adrese" task.
  - The skip list covers calendar invites, newsletters, T&C updates, marketing.
  - Non-invoice mail: store only the ID and "skipped". **Never store or summarise its content.**
- **Mailbox export upload:** the Upload page also accepts a Google Takeout `.mbox` file (one-time history import for mailboxes that are only forwarded). Each message goes through triage exactly like live mail.
- **Invoices that are only in the email body** (Canva, Stripe/Anthropic, Bolt, Wolt, Venden "paid" notices): render the body to PDF and treat that PDF as the document.

## B. WhatsApp (Phase 5)

The team's existing group chat can't be connected officially. Options, in order:

- **A. The app's number creates a new group** via the Groups API. Requires Meta's Official Business Account status; at most 8 participants; webhook for every photo or PDF.
- **B. A one-to-one receipts number.** Needs only App Review.
- **C. A monthly chat export** (.zip "with media") uploaded on the Upload page. Always available.
- **Never** use unofficial WhatsApp-Web libraries; the number can be banned.

For A and B:
- Download the media **immediately** (webhook media IDs expire after 7 days; URLs after about 5 minutes).
- The sender is only a hint: the card number on the receipt and Gabriels's confirmation decide the entity.
- A caption becomes `purpose`.
- Replies go only to the sender, inside WhatsApp's 24-hour window, in Latvian: "Saņemts: Bolt 12.40 EUR, gaida apstiprinājumu" / "Saglabāts: BZ-2026-00123".

## C. Document processing (worker)

1. **Hash** (SHA-256). If the hash is known, link the new source to the existing document and stop.
2. **Convert:**
   - `.doc`/`.xls` → LibreOffice → PDF/XLSX;
   - `.edoc` → unzip, process the inner document, keep the container;
   - images → OCR (`lav+eng`);
   - PDFs with a garbled text layer (shifted-font encodings, e.g. the GRIFS invoice) → render the pages and OCR them.
3. **Extract** all **E** fields of `03` (including item lines and VAT lines). Validate:
   - net + VAT = gross (±0.01);
   - the VAT lines sum to VAT;
   - the dates are plausible;
   - reg. no. (11 digits) and VAT no. (LV + 11; EU formats);
   - EU VAT no. → VIES check, result stored.
4. **File-name check.** If the invoice no. in the content ≠ the number in the file name → `filename_mismatch`; the content wins (e.g. the Cemme file named C062510 that contains C082506).
5. **Versions.** Same seller, same number, different amount (Rēzekne 250023: 749.99 vs 750.00) → `version_of`, and a decision card (D-DOC-1).
6. **Attach supporting documents** to their transaction instead of creating new transactions: order confirmations, delivery notes, reminders, debt letters, reconciliation acts (Euronics order 115833603 → invoice PR25-047028).
7. **Build or extend the transaction record** (`03`) and compute every **R** field: entity suggestion, category, accounts, VAT regime, deductible share, project/grant suggestion, accrual flag, asset flag, booking deadline.
8. **Entity suggestion order** (shown with its reason):
   1. buyer reg./VAT no.;
   2. buyer name;
   3. MNB → "MNB Expense", not booked;
   4. private person as buyer → decision D-ENT-2;
   5. card number on the receipt;
   6. the matched bank line's account;
   7. no suggestion.
9. **Status `awaiting_decision`.** The record waits for Gabriels's confirmation (and any decision cards). **Nothing is filed, posted or written to a Sheet before confirmation.**
10. **After confirmation:** rename to `YYYY-MM-DD_Supplier_Gross_DocNo.ext` (issue date; `/` → `-`; statements and letters use a descriptor; unreadable no. → `invoice-no-unreadable`), file in `<entity root>/<YYYY> Finances/<category>/<supplier?>/`, then post (`06`), write the Sheet, run the controls. If the entity is changed later: reverse the posting, re-post, move the file and the Sheet row, with an audit entry.

**Category folders** (they already exist in the 2025 Drive; each maps to a default account in `06`):

| Folder | Covers |
|---|---|
| Telpu noma | rent |
| Grāmatvedības pakalpojumi | accounting services |
| Video pakalpojumi | video services |
| Pasākuma izdevumi/<supplier> | event costs |
| Pasākuma izejmateriāli | event materials |
| Materiālu iegāde | material purchases |
| Mārketinga izdevumi | marketing |
| 6110 | issued invoices |
| 7170/<supplier> | transport |
| 7550/<supplier> | services and subscriptions |
| 7720/<supplier> | equipment |
| 8260 | letters, reminders, reconciliation acts |
| MNB Expense, Kanango | out of scope |

**The numbers are folder labels, not ledger accounts.**

## D. Bank import

- **Weekly reminder,** Monday 09:00, per account, showing the last upload date. Red after 10 days.
- **Formats** (auto-detected):
  - Swedbank CSV (sample in `fixtures/bank/`);
  - Luminor CSV (sample from Gabriels);
  - FiDAViSta XML;
  - camt.053.
- **The IBAN in the file decides the account.** A file belonging to the other entity is refused.
- **Balances must tie out.** The opening balance of an import must equal the closing balance of the previous one, and the sum of lines must equal closing − opening. Otherwise the import is refused with the difference shown. Opening and closing balances come from the file's balance records (Swedbank CSV: record type 10 = opening, closing types per the bank's format description; FiDAViSta/camt.053: balance elements). If a file has no closing balance, the upload screen asks Gabriels to type it from the statement; it is never assumed.
- **Dedupe** on `(account, booking_date, amount, details_hash)`.
- **Card transaction date:** parsed from the details ("PIRKUMS …7665 30.12.2024"). It is used for matching and for the economic period.

## E. Automatic bank statuses

| Pattern | Result |
|---|---|
| Bank and card fees, interest | auto-transaction `bank_only`, posted under the fee rule (no decision) |
| Payments to the VID tax account | auto-transaction against tax liabilities |
| Own-account transfers (incl. GES current ↔ grant) | auto internal transfer; the grant tag carries over |
| Transfers between Bourzma, GES and MNB | `internal` → decision D-ENT-3 |
| Lender per `loans` | loan movement |
| ATM withdrawal or cash deposit | cash-book entry |
| Incoming without an issued invoice | `income_needs_document` → decision D-INC-1 |
| Spend without a document | `missing` → request draft, or decision D-DOC-2 |

## F. Matching (each match stores method and confidence)

1. **Reference.** The invoice no. parsed from the details (diacritics stripped: "R ins Nr. C072512", "Rek.Nr.250004") ↔ a transaction of the same entity and counterparty.
2. **Amount + date + counterparty.** Exact amount, fuzzy name or IBAN, document date 0–60 days before (up to 365 for prior-year items, which set the prior-year period).
3. **Card day-sum.** Card lines grouped by card + transaction date, matched to that day's receipts from the same merchant. CityBee pre-charges 5.29 and then charges the remainder: 30.12.2024 → 5.29 + 5.29 + 52.61 = 63.19 = 5.29 + 1.90 + 5.29 + 45.42 + 5.29.
4. **Partial and multiple payments.** Burka 1573/07-2025: 677.60 + 290.40 = 968.00. One payment may cover several invoices.
5. **Entity from the match.** For a record with no buyer and no card number, the matched account's entity becomes the pre-selected suggestion (`bank_account`). Gabriels still confirms.
6. **Duplicates.**
   - Same counterparty + amount + reference twice, or the same amount within 120 days with only one invoice → decision D-DUP-1.
   - Known cases: A-Sound 07/04 685.67 ×2, GD&L V25-06-01 605.00 ×2, income SKR 250002 605.00 ×2.
7. **Debt letters and reconciliation acts** are linked to the underlying transactions. The creditor's balance is compared with the ledger, and the difference goes to a decision card. Claimed fees and penalties are kept separate from principal (D-DEBT-1).
