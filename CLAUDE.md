# accounting-legal-panel: self-running bookkeeping and legal management for Bourzma + Green Echo Studio (+ MNB Production, legal only)

The app **keeps the complete double-entry books** of two Latvian entities, without an outside accountant doing the work (an appointed bookkeeper reviews and signs off; see `docs/01` legal gate 1):
- it captures every document and bank transaction;
- it records each one in a full **transaction register**;
- it posts it to the general ledger;
- it keeps the VAT, payables/receivables, assets, loans and cash registers;
- it runs its own control checks;
- it produces filing exports (PVN, paziņojums) and annual-report drafts for filing in EDS by a person.

A second section, **Legal & Authorities** (`docs/12`), manages contracts, letters from VID and other authorities, disputes and debt claims for Bourzma, Green Echo and MNB Production, with deadlines in Google Calendar and replies drafted in Latvian.

**Gabriels makes the decisions** in a fixed list of decision areas (`docs/05-decisions-catalog.md`). Everything else is automatic, rule-based and cited to Latvian law.

| | Bourzma (`BOURZMA`) | Green Echo Studio (`GES`) |
|---|---|---|
| Legal | Biedrība "Bourzma – Radošas Ilgtspējības Platforma", reg. 40008334474, VAT LV40008334474 | SIA "Green Echo Studio", reg. 40203682026, VAT LV40203682026 |
| Books / PVN period | Double-entry / **quarterly** | Double-entry / **monthly** |

SIA "MNB Production" (40203389163) is out of scope for the books (folder "MNB Expense", never booked) but **in scope for Legal & Authorities**. **All three share one address. Never route by address.**

## Core flow (Phase 1 delivers it end to end)

```
1  Document arrives (gabrielsk@, forwarded info@ / CityBee; later WhatsApp) ─┐
   Bank statement uploaded (weekly) ──────────────────────────────────────────┤
2  App extracts every field and builds ONE TRANSACTION RECORD (docs/03)  <────┘
3  App suggests: entity, category, account, VAT treatment, project/grant, match
4  GABRIELS CONFIRMS: one click (or "confirm all ≥95 %"), or answers the decision card
5  App, in one database transaction (external steps a and e run through the outbox, retried until done; see docs/02):
     a. files the renamed document in Drive  <entity>/<YYYY> Finances/<category>/
     b. writes the full record to the TRANSACTION REGISTER
     c. posts the journal entry to the GENERAL LEDGER
     d. updates the sub-registers: VAT, payables/receivables, bank, assets, loans, cash, grants
     e. re-runs the control checks and refreshes the dashboard and the entity's Google Sheet
```

## Hard rules

1. **Never act outside the app's own data.** No logging in to EDS, banks or Moneo. No submitting. The only sending allowed:
   - WhatsApp replies to the team member who sent a receipt;
   - creating and updating events in Gabriels's "Juridiskie termiņi" Google Calendar;
   - sending short events (no personal codes, IBANs or document text) to Gabriels's personal-assistant hub (`docs/13`);
   - nothing else (supplier emails are drafted for Gabriels to send).
2. **Gabriels decides, the app prepares.** Anything in the decision catalog needs his answer. The app always offers a recommended answer with the legal source. Everything outside the catalog follows `rules`, automatically.
3. **Never guess a value.** Unreadable → `null` + `unreadable_fields` + a decision card.
4. **Never delete or overwrite.** Originals are kept (SHA-256, source ID). Posted entries are never edited: corrections are reversal + new entry, with an audit trail.
5. **Everything traces back.** Every ledger line, register row, VAT figure and report figure links to its transaction record → document(s) → bank line(s).
6. **Law as data.** Rates, thresholds, deadlines, form layouts, XSD schemas and retention periods live in `rules` with a source URL and `verified_at`. Warn after 90 days. Never hard-code a tax fact.
7. **Locked periods are locked.** After a VAT period is filed or a month is closed, postings into it are blocked; changes go into the open period as corrections.
8. **Data:** internal tool. Store full IBANs, card numbers and personal codes, with no masking. Login is limited to an allow-list; secrets go in env vars or Supabase Vault.
9. **Language:** the UI is in English with Latvian legal terms in brackets. Anything for VID, suppliers or the board is in Latvian.

## Stack

Next.js (Vercel; UI + webhooks only) · Supabase (Postgres = source of truth, Auth, Storage, pg_cron, `jobs` queue) · **Worker container** (Railway/Fly.io: Gmail, conversions with LibreOffice/tesseract/edoc, Claude, Drive, Sheets, parsing, posting, checks, filing exports) · Google Gmail/Drive/Sheets/Calendar APIs · Anthropic API · WhatsApp Cloud API (Phase 5).

## Spec files. Read only what the current task needs.

| File | Covers |
|---|---|
| `docs/01-entities.md` | Entities, accounts, cards, people, legal set-up and legal gates |
| `docs/02-architecture.md` | Services, data model, AI models and costs |
| `docs/03-transaction-register.md` | **The transaction record: every field, how each is filled, lifecycle** |
| `docs/04-capture-and-matching.md` | Email, WhatsApp, file processing, bank import, matching, Drive filing |
| `docs/05-decisions-catalog.md` | **Exactly what Gabriels decides**, with defaults and sources |
| `docs/06-ledger.md` | Chart of accounts, posting engine, sub-registers, period close, year-end |
| `docs/07-vat-and-filings.md` | PVN, filing exports (Excel now, EDS XML optional), payments to private persons, calendar |
| `docs/08-controls.md` | Automatic control checks that keep the books right without an accountant |
| `docs/09-dashboard.md` | UX: screens, flows, design rules for a smooth, clear dashboard |
| `docs/10-migration.md` | 2025 import, 2026 backfill, Green Echo history, opening balances |
| `docs/11-phases-and-tests.md` | Build order, acceptance criteria, golden tests |
| `docs/12-legal-and-authorities.md` | **Legal & Authorities:** contracts, VID and other authority letters, disputes and debt claims (incl. MNB), deadlines in Google Calendar, drafted replies |
| `docs/13-ecosystem-events.md` | One-way events to Gabriels's personal assistant hub (deadlines, decisions, urgent letters); shared contract for all his apps |
| `docs/diagrams.md` | Mermaid diagrams: parts and connections, invoice flow, bank matching, VID letter flow, record lifecycle, monthly routine |
| `SETUP.md` | Step-by-step setup for Gabriels (accounts, keys, deploy, routine) |

## How to work

- Build phase by phase (`docs/11`). Use plan mode first. Tests first for parsers, the posting engine, VAT maths and the control checks.
- After each phase: `reports/PHASE_n.md` and a demo to Gabriels.
- If a spec is unclear or contradicts another, ask Gabriels; never invent a policy.
- Commands: `pnpm dev`, `pnpm test`, `pnpm db:migrate`, `pnpm worker:dev`. Keep them working and list new ones here.
