# 10: Migration and opening balances

## A. Import the 2025 work already done
- **Source:** `fixtures/Bourzma_2025_tracker_and_invoice_register_v9.xlsx`. It includes the tracker, the invoice register, the open items at 31.12.2025 and the opening items at 01.01.2025.
- **Process:**
  1. Each tracker row becomes a transaction record with its existing status.
  2. Gabriels's past decisions become `decisions` rows: Pittore is personal, Euronics is a company expense, Dokobit is OK, and Burka was paid in two parts (677.60 + 290.40).
  3. Flags carry over: the "relates to 2024" flag, SKR 250002 received twice, A-Sound and GD&L paid twice, the VEF Kvartāls debt (3,445 principal + 5,717.25 in claimed fees), and the 60 income lines with no issued invoice.
- **Result:** the imported items enter the decision queue as cards. They are not posted automatically.

## B. Back-fill from the sources
1. Import all 2025 and 2026 bank statements (Swedbank, Luminor).
2. Run the Gmail sweep on gabrielsk@ for 2024-12 to today. Forwarding only covers mail from the day it is switched on, so also import a **one-time Google Takeout export (.mbox) of info@bourzma.com** (and of spended0@ for CityBee) for the same period on the Upload page; it goes through the same pipeline and dedupes by SHA-256 and message ID.
3. Index the existing Drive folders (`2025 Finances`, `2026 Finances`). Existing files are linked, never moved or renamed.
4. The matching engine runs over everything, and conflicts become cards.

## C. Opening balances (per entity)
- **Bourzma, at 01.01.2025:**
  - closing figures of the 2024 gada pārskats (from the Project or Gabriels);
  - bank balances from the statements;
  - opening payables and receivables (e.g. the 2024 VEF invoices).
- **Green Echo, from its start date or 01.01.2025:** the same sources.
- **Check:** the opening trial balance is in balance, and the bank opening balance equals the statement.
- **Approval:** Gabriels approves the opening balance as a decision card. Periods before it stay locked.

## D. Order
A → C → B. The 2025 year is closed first, with its annual-report draft prepared, then 2026 is brought up to date.
