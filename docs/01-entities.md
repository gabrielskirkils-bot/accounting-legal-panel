# 01: Entities, accounts, cards, people and legal set-up

## Entities

| Field | Bourzma (`BOURZMA`) | Green Echo Studio (`GES`) |
|---|---|---|
| Legal form | Biedrība (association), partly grant-funded | SIA (limited company) |
| Reg. no. / VAT no. | 40008334474 / LV40008334474 | 40203682026 / LV40203682026 |
| Bookkeeping | Double-entry | Double-entry |
| PVN period | Calendar quarter | Calendar month |
| Annual report | MK noteikumi Nr. 439, double-entry variant | Gada pārskatu un konsolidēto gada pārskatu likums |
| Drive root | `BOURZMA - BIEDRĪBA/ACCOUNTING/FINANCES/` (ID set in Phase 0) | Created or chosen in Phase 0 |

Out of scope: SIA "MNB Production" (40203389163). Its documents go to "MNB Expense" and are never booked.

## Bank accounts

| Entity | Bank | IBAN | Purpose | Ledger account |
|---|---|---|---|---|
| BOURZMA | Swedbank | LV70HABA0551057252619 | current | bank – Swedbank current |
| BOURZMA | Swedbank | LV83HABA0551061587024 | credit | bank – Swedbank credit (negative balance = liability) |
| GES | Luminor | LV54UNLA0055005604951 | current | bank – Luminor current |
| GES | Luminor | LV13UNLA0055006268781 | **grant** | bank – Luminor grant |

## Cards. The card decides the entity and the account, never the person.

| Card | Holder | Entity | Account |
|---|---|---|---|
| …1187 | Gabriels Kirkils (sometimes used by Patriks) | BOURZMA | LV70HABA0551057252619 |
| …7665 | Patriks Kirkils (CEO) | BOURZMA | LV70HABA0551057252619 |
| …7441 | Kristaps (project/construction manager) | BOURZMA | LV70HABA0551057252619 |
| 5148360002467934 | Gabriels Kirkils | GES | LV54UNLA0055005604951 (confirm in Phase 0) |

The full Bourzma card numbers are read from the statements and confirmed by Gabriels in Phase 0.

## People

Gabriels Kirkils, Patriks Kirkils, Kristaps, Terēze Peipiņa, Mārtiņš Kirkils (lender, 19.12.2024), Kaspars Mārtiņkrists. Each person has a personal code, IBAN, phone, email and roles (board member of which entity, card holder, lender, author …). Gabriels supplies them in Phase 0; never guess them. Authors, prize winners and private customers are added automatically when they appear.

## MNB Production (Legal & Authorities only)

SIA "MNB Production", reg. 40203389163. Not booked here. Needs its own Drive root (for `LEGAL/`) and appears in the entity switcher of the Legal screen only.

## Legal gates. Settle these in Phase 0, before the app is used as the only bookkeeping.

The Grāmatvedības likums decides **who may keep the books**. Store the articles in `rules` and show these gates in Settings → Legal set-up:

1. **Who keeps the books (Art. 34–35).** Books are kept by a bookkeeper or a licensed outsourced bookkeeper under a written agreement. Art. 35 lists who may keep the books **personally**. It includes:
   - a **board member of an association** (applies to Bourzma if Gabriels is on Bourzma's valde);
   - the **sole board member of a capital company who is also its sole shareholder** (applies to Green Echo only if Gabriels is GES's sole board member *and* sole owner).

   **Decided (19.11.2026): a bookkeeper will be appointed.** Build the app exactly as specified: it does all the work, and Gabriels makes the decisions in `05`. The appointed bookkeeper is recorded per entity in Settings → Legal set-up (name, agreement date, which entities) and gets the `bookkeeper` role: they can view everything, review, post corrections and sign off a closed month or year. Seed both entities as `bookkeeper_under_agreement` with the name left empty until Gabriels enters it; an empty name is an amber control, never a blocker. The other modes stay available:
   - `self_kept_by_board_member`
   - `bookkeeper_under_agreement` (name, agreement date)
   - `licensed_outsourced` (licence no.)
2. **Accounting organization documents (grāmatvedības organizācijas dokumenti).** Each entity needs its own documented accounting rules:
   - chart of accounts;
   - document flow;
   - inventory procedure;
   - valuation and capitalisation threshold;
   - depreciation;
   - grant accounting;
   - cash rules.

   The app generates a **draft in Latvian** from the settings chosen in `05-decisions-catalog.md` (D-POL-*). Gabriels approves it, and it is stored as a document with a version history.
3. **Recording deadlines (Art. 13).** The general rule is to record documents no later than 20 days after the end of the month in which they were received or issued. Some associations have a quarterly rule. Verify which applies to each entity and store it in `rules`. The control checks (`08`) flag unrecorded documents before the deadline.
4. **Retention (Art. 28).** Accounting registers and organization documents: 10 years. Supporting documents: at least 5 years, and as long as needed to trace transactions. Annual reports: until the entity is reorganised or wound up. The app shows each item's "keep until" date, and Drive files are never deleted.
5. **Signing.** Annual reports are signed by the board (and, if applicable, the bookkeeper) in EDS by a person, never by the app.
