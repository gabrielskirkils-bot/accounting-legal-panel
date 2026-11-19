# 11: Phases and tests

Each phase ends with its tests green and a short demo to Gabriels. **The core flow runs from Phase 1**, and every later phase plugs into it.

## Phase 0: setup (1–2 days)
- Repo, Supabase, the worker container, auth allow-list, env vars, Drive/Sheets/Gmail OAuth.
- A helper `pnpm google:auth` that opens the Google consent screen for gabrielsk@ and prints the refresh token for `.env` (scopes: gmail.readonly, drive, spreadsheets, calendar.events).
- `.env.example` with every variable from `02`, and a `SETUP.md` check: `pnpm doctor` verifies each key works (Supabase, Anthropic, Gmail, Drive roots, Sheets) and prints what is missing.
- Seed `entities`, `bank_accounts`, `cards`, `people` from `01`.
- Seed `rules` with **looked-up** values and source URLs (PVN rates, itemisation threshold, deadlines, cash limits, private-person tax, retention). Never typed from memory.
- Policy wizard: answers D-POL-1…10 → draft accounting organization documents (PDF, LV) for Gabriels to sign.
- Enter the appointed bookkeeper's name when known (Settings → Legal set-up); not a blocker.

## Phase 1: core flow (email → record → decision → atomic post)
- Gmail capture (gabrielsk@ + forwarded info@ and spended0@), file conversion, OCR, extraction, VIES.
- Transaction register (`03`), decision cards D-ENT-1/2/3, D-DOC-1 (`05`).
- On confirm, atomically: Drive copy + rename, register, journal entry (`06` templates), sub-registers, Sheet row.
- Today + Register screens (`09`).
- Tests: G1–G14, posting templates, atomicity (kill the worker mid-post → nothing half-done), idempotency (same email twice → one record).

**Gabriels uses this daily from the end of Phase 1.**

## Phase 2L: Legal core (runs alongside Phase 2)
- Legal inbox (email routing, EDS/e-adrese tasks, Drive `_Ienākošie` watch), matters and contracts registers, deadline calculation from `rules`, Google Calendar sync, the Legal screen, controls C24–C30, escalation flags, the start-up load and seeding of known matters (`12` section 10).
- Ecosystem events (`13`): `events_out` + sender; with `EVENTS_HUB_URL` empty they are stored only.
- Drafting (assessment, plan, Latvian letters, answer packs from the books, lawyer brief) lands in Phase 3, once the ledger can supply the figures.
- Tests: L1–L8 below.

## Phase 2: bank and matching
- Swedbank/Luminor CSV, FiDAViSta, camt.053 import with balance tie-out (C2, C3).
- Matching (reference, amount+date, card day-sum, partial, duplicates), D-DOC-2, D-INC-1, D-DUP-1, D-DEBT-1.
- Money screen; Monday upload reminder.
- Tests: B1–B6; statement continuity; 1-cent rounding cases.

## Phase 3: ledger complete + VAT
- Sub-registers (fixed assets, loans, cash, grants, donations), depreciation, accruals, FX, month close (D-CLOSE-1), period locking.
- VAT registers, partial deduction (D-VAT-3), reverse charge, PVN 1 export (Excel; XML writer optional, see `07`) (D-VAT-2).
- Controls C1–C23 nightly (`08`). Taxes screen.
- Migration A → C → B (`10`).
- Tests: trial balance = 0 after migration; PVN for one real 2025 Bourzma quarter reproduces by hand-check; locked period refuses posting.

## Phase 4: year end, reports, filings
- Year-end proposals (D-YEAR-1), inventory lists, reconciliation acts (LV), annual-report drafts (MK 439 for Bourzma; GES by size), paziņojums (D-PRV-1/2), filing calendar + reminders, accountant pack export.
- Reports screen.
- Settings → Health (`09`).
- Tests: 2025 annual-report draft for Bourzma ties to the trial balance; comparatives = last year's report.

## Phase 5: WhatsApp + hardening
- WhatsApp capture (option chosen in `04`), mobile PWA polish, push.
- Audit log review, failed-jobs screen with retry, AI budget stop, nightly DB backup to Drive, capture-health alerts, posting-engine self-check.

## Golden test set (`fixtures/golden/`, each with `expected.json`)

Gabriels copies these files from Drive into `fixtures/golden/` in Phase 1. They are all in the 2025 Drive folders.

| # | Case | Expected |
|---|---|---|
| G1 | GRIFS AG invoice, PDF with garbled embedded font | OCR fallback; invoice no. 2025075835, date 14.07.2025, net 10.00, VAT 2.10, gross 12.10, BOURZMA |
| G2 | Cemme .xls named "…(C062510)" that contains C082506 | `filename_mismatch`; invoice no. taken from the content; gross 134.92 |
| G3 | Bauska .edoc (contract BKC/2025/1-14.3.2/6) | unzipped; `doc_type = contract`; 1,000.00; customer Bauskas novada pašvaldība |
| G4 | Wolt receipt 2.09, buyer MNB Production | D-ENT-1 suggests MNB (not booked); no posting |
| G5 | Pittore BB081225_8, 6.90, buyer Patriks Kirkils | opens D-ENT-2 (private-person buyer) |
| G6 | Dokobit PO250910624, 9.00, LT supplier, 0 % VAT | `vat_regime = reverse_charge (EU service)`; PVN 1 Part II |
| G7 | Rēzekne 250023: two versions, 749.99 and 750.00 | linked with `supersedes_document_id`; `two_versions`; the bank 750.00 matches the 750.00 version |
| G8 | Euronics PR25-047028 + order confirmation 115833603 | the order is `order_confirmation`, linked, not counted; one posting only |
| G9 | Burka 1573/07-2025, 968.00 | matched to two bank lines (677.60 + 290.40); balance 0 |
| G10 | Julianus/VEF letter 3186319 | `debt_letter`; principal 3,445.00 and claimed fees 5,717.25 kept separate; linked to VK-24/0441 and VK-24/0452 |
| G11 | Zīle-Zīle akts 2/2025 (.doc) | LibreOffice conversion; 550.00; no VAT shown → VAT `null`, not 0 |
| G12 | Canva invoice email (body only, no attachment) | body rendered to PDF; reverse charge |
| G13 | Forwarded info@ email (LMA 003813) | `original_sender` = lma.lv; date = the original date |
| G14 | Green Echo receipt paid with card 5148360002467934 and no buyer printed | entity GES by card |

Bank cases (`fixtures/bank/`):

| # | Case | Expected |
|---|---|---|
| B1 | CityBee 30.12.2024 card lines vs 3 invoices | day-sum match, 63.19 = 63.19; `relates_to_year = 2024` |
| B2 | A-Sound 07/04 paid on 11.04 and 05.06 | duplicate flagged |
| B3 | GD&L V25-06-01 paid on 16.06 and 05.09 | duplicate flagged |
| B4 | SKR 250002 received twice (03.02, 13.02) | income duplicate flagged |
| B5 | A GES statement uploaded under BOURZMA | refused |
| B6 | Bauska: 200.00 (ref 250004) + 800.00 (ref 250007) | both linked to the Bauska contract and invoice; gap 250004 flagged in the numbering check |

Unit tests are also required for: parsers, the VAT maths (rounding at line and document level), the tile roll-up (counts add up), and the routing rules (D-ROUTE-1 in `05`).

Additional required tests:
- **Ledger:** every template in `06` balances; reversal + repost equals the corrected result; closing the year carries forward correctly.
- **VAT:** registers = ledger VAT accounts; itemisation threshold from `rules`; reverse charge nets to zero when fully deductible; Bourzma partial deduction at the provisional and final rates.
- **Controls:** each of C1–C23 has a failing fixture that turns it red/amber and a fixed one that turns it green.

Legal cases (`fixtures/legal/`, Gabriels supplies the files):

| # | Case | Expected |
|---|---|---|
| L1 | EDS notification email | matter in `fetch_from_eds`; calendar event today; C25 red next working day if not done |
| L2 | VID information request PDF with "15 dienu laikā" | deadline computed from the receipt rule in `rules`, basis shown; internal date = −3 working days; two calendar events |
| L3 | Extension granted (new letter, same reference) | linked to the same matter; deadline and events updated, not duplicated |
| L4 | Julianus/VEF letter 3186319 | matter `debt_claim_against_us`; principal and fees separate; linked to VK-24/0441, VK-24/0452 and the G10 document |
| L5 | Bailiff or court document | `escalation_required`, red banner, lawyer brief generated, drafts locked as DRAFT |
| L6 | Signed contract `.edoc` with a notice period | signers and validity read; notice deadline computed; D-CON-2 card 30 days ahead |
| L7 | Letter addressed to MNB Production | entity MNB in Legal; nothing posted in the books |
| L8 | VID request for invoices of a period | answer pack lists exactly the matching transactions and documents; anything missing is listed as missing |
