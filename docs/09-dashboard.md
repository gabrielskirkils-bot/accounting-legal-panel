# 09: Dashboard. Smooth, fast and clear.

**Principles:**
1. **One next action** is always at the top.
2. **Decisions come one at a time**, never as a wall of forms.
3. **Every number is clickable** down to the document.
4. **Nothing waits on a spinner:** optimistic UI plus realtime.
5. **Same layout on desktop and mobile;** decisions are thumb-friendly on phone.

**Stack:**
- shadcn/ui + Tailwind.
- TanStack Table (virtualised) for the registers.
- Recharts for the few charts.
- Supabase realtime for live updates.
- An entity switcher in the header: **Bourzma | Green Echo | Both**. The colour accent changes with the entity, so it is always obvious which books you are in.

## Screens (6, plus Settings)

### 1. Today (home)
- **Top bar.**
  - "Next: confirm 4 documents (≈2 min)", as a single button.
  - Status tiles for each entity:
    - waiting for decision;
    - unmatched bank lines;
    - payables due within 7 days;
    - receivables overdue;
    - next filing and days left;
    - red controls.
- **Decision queue.** One card at a time.
  - **Left:** the document preview (PDF/image, zoomable).
  - **Right:**
    - the extracted fields, with low-confidence ones highlighted;
    - the suggestion and the recommended option;
    - the result preview: "Will post: 7170 Transport 24.79 / PVN 5.21 / Swedbank 30.00 → Drive /2025 Finances/7170/…".
  - **Keys:**
    - `Enter` confirms the recommended option.
    - `1`–`5` pick another option.
    - `E` edits a field.
    - `S` skips.
    - `B` sends it back for re-extraction.
- **Batch mode:** "Confirm all ≥ 95 % (12)" shows a list review first, then one click.
- **Filing strip:** the next 3 deadlines.

### 2. Register
- One table of all transaction records:
  - columns from `03`;
  - saved views: *This month*, *Waiting*, *Unpaid*, *Personal*, *Grants*, *VAT period*;
  - full-text search across supplier, number and amount.
- **Row click → side panel:**
  - document;
  - bank match;
  - journal lines;
  - decision history;
  - audit trail;
  - a "Reverse" button.
- Export the current view to CSV or Excel.

### 3. Money
- **Bank tab:** the bank lines per account, with matched and unmatched states, a drag-to-match control, and the statement balance vs the ledger balance.
- **Payables / Receivables tab:** ageing buckets, each with a "Draft reminder (LV)" action.
- **Cash and loans tabs.**
- **Weekly upload:** drag a statement onto the page. The import shows "76 lines, 71 matched automatically, 5 need you".

### 4. Taxes
- The VAT period cards. Each shows:
  - the draft PVN 1 figures with a drill-down;
  - the checklist from `08`;
  - **Approve → Download export**;
  - a field for the EDS number.
- The paziņojums.
- The filing calendar.

### 5. Reports
- Profit and loss, and balance sheet, per entity and period, with comparatives.
- Grants: budget vs actual per line.
- Projects: margin per project.
- The annual-report draft.
- Trial balance and general ledger (virsgrāmata).
- **Accountant pack:** all of the above as one Excel file, for the appointed bookkeeper or an audit.

### 6. Legal
Matters, contracts and authorities, as specified in `12` section 9. Legal deadlines also appear in the Today filing strip, so there is one queue.

### Settings
- Entities, accounts and cards.
- Chart of accounts.
- Policies (D-POL-*).
- Rules, with their source links.
- Legal set-up (bookkeeper per entity) and Schemas (optional VID XSDs).
- Integrations: Gmail, Drive, Sheets, WhatsApp.
- Health: a green/red light per integration (Gmail token, Drive roots, Sheets, Anthropic, worker heartbeat, last forwarded mail per address, last bank upload per account), the same checks as `pnpm doctor` and control C18.
- Users and access (Gabriels and Patriks: everything; bookkeeper: books + legal matters linked to the books; a lawyer: one matter).
- AI budget.

## Speed targets

- Today screen: first paint < 1 s.
- Next decision card: loads in < 300 ms, because the next 3 are prefetched.
- Registers: 50k rows, smooth scrolling.
- Confirm: the UI updates instantly; the posting job finishes within 5 s, with a toast on failure and an undo.

## Mobile

- A PWA, installable.
- Today and the decision cards are fully usable on a phone.
- A camera upload button for receipts.
- Push notifications for new decisions and deadlines.

## Language

- The UI is in English.
- Latvian terms are shown where the law names them (PVN, paziņojums, gada pārskats, kreditori and so on).
- All letters to suppliers, VID and counterparties are in Latvian.
