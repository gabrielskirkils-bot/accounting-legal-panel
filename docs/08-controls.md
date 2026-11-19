# 08: Controls. How the app proves it is right.

**Schedule:**
- **Nightly, via pg_cron:** every control below runs, and the results go to `control_results`.
- **Before month close and VAT approval:** the same set runs again, and it must be green, or each warning must be accepted with a reason.

**Results:**
- A red control blocks closing.
- An amber control shows on the Today screen.
- Each result links to the records that caused it.

| # | Control | Rule | Level |
|---|---|---|---|
| C1 | Trial balance | debits = credits, per entity and period | red |
| C2 | Bank tie-out | ledger bank balance = statement closing balance, per account and date | red |
| C3 | Statement continuity | each import's opening balance = the previous closing balance; no date gaps | red |
| C4 | VAT tie-out | the VAT registers = the VAT accounts; the draft PVN 1 = the registers | red |
| C5 | Sub-ledgers | payables, receivables, fixed assets, loans, cash and grants each equal their control accounts | red |
| C6 | Numbering | `txn_no` and issued invoice numbers have no gaps or duplicates | red |
| C7 | Unmatched bank lines | older than 14 days | amber → red at the booking deadline |
| C8 | Booking deadline | records not posted within the Art. 13 deadline | red |
| C9 | Unpaid documents | past due date: payables and receivables ageing | amber |
| C10 | Duplicates | same supplier + amount + date ±3 days, or the same document hash | amber (opens D-DUP-1) |
| C11 | Wrong entity | buyer on the document ≠ booked entity | red |
| C12 | Missing purpose | representation, private-person payments and personal-card items without purpose or evidence | amber |
| C13 | Grant over-spend | spending on a budget line > budget, or outside the grant period | red |
| C14 | Cash limits | cash balance or a single payment over the `rules` limits; negative cash | red |
| C15 | VIES | an EU VAT number not checked, or checked as invalid | amber |
| C16 | Personal items | personal amounts not repaid after 30 days (D-POL-9); a deemed-distribution risk for GES | amber |
| C17 | Intercompany | the Bourzma ↔ GES balances mirror each other | red |
| C18 | Capture health | no forwarded email in 7 days; the bank upload is late (Monday); WhatsApp media download failures | amber |
| C19 | Rules freshness | a `rules` row whose source has not been rechecked in 90 days (CLAUDE.md rule 6), or whose `valid_to` has passed | amber |
| C20 | Filing calendar | a filing due within 7 days and not ready | amber → red at 2 days |
| C21 | Sheets mirror | the row count and totals in the Sheet = the database | amber (auto-resync) |
| C22 | Drive | every confirmed document has its Drive file ID and the file exists | red |
| C23 | AI spend | monthly AI cost > 80 % of the budget | amber |
| C24–C30 | Legal & Authorities controls | see `12` section 8 |

**Self-check of the posting engine:** a weekly job re-posts a sample of 50 confirmed transactions in a dry run and compares the result with the stored entries. Any difference is red.
