# 02: Architecture, data model, AI models and costs

## Services

```
Gmail (gabrielsk@, incl. forwards) ─poll 15 min─┐
WhatsApp Cloud API (Phase 5) ────webhook────────┤→ Next.js route: verify, store raw, enqueue
Uploads (bank files, docs, chat export) ────────┘
                         │
              Supabase Postgres: jobs queue (idempotent, retries, failed screen)
                         │
   WORKER (Docker, Railway/Fly.io) — all heavy work:
     fetch · convert (LibreOffice, tesseract lav+eng, edoc unzip, HTML→PDF via Playwright)
     · extract (Claude) · validate · build transaction · suggest · match
     · post (after confirmation) · file in Drive · write Sheet · run controls · build filing exports
                         │
   Next.js UI (Vercel) ←→ Supabase (realtime subscriptions for instant dashboard updates)
```

- Nothing heavy runs on Vercel or in Edge Functions (no LibreOffice or tesseract there, and they time out).
- Jobs are idempotent (keyed by SHA-256, Gmail message ID, WhatsApp message ID or bank-line key). They retry with backoff (1 min, 10 min, 1 h), then `failed`, which is visible on the dashboard.
- **Posting is atomic.** Steps 5a–5e of the core flow run in one DB transaction plus an outbox: Drive and Sheet writes are retried from the outbox, so the books never depend on Google being up.
- pg_cron (pg_cron runs in UTC; every time below is **Europe/Riga** and must be converted, including the summer/winter time change. Simplest: schedule hourly ticks and let the worker decide by Riga local time):
  - Gmail poll: every 15 min;
  - control checks: nightly at 02:00;
  - deadlines: daily at 08:00;
  - bank-upload reminder: Monday 09:00 Europe/Riga;
  - Sheet notes read-back: hourly;
  - rules freshness: weekly;
  - Drive `LEGAL/_Ienākošie` watch: every 15 min;
  - Google Calendar sync check (C26): nightly.

## Auth and roles

- Magic link, allow-list only.
- `owner` (Gabriels): everything, including decisions, period close and EDS file export.
- `member` (Patriks, Terēze): view, upload, add purpose and participants on their own receipts.
- `bookkeeper`: the appointed bookkeeper (`01` legal gate 1); can view everything, review, post corrections and sign off closed periods. Cannot change settings or policies.
- RLS: allow-listed users can read; writes are checked against the role.

## Data model (core tables)

```
-- master data
entities(id, code, name, reg_no, vat_no, vat_period, legal_form, bookkeeping_mode, bookkeeper_name, bookkeeper_agreement_date,
         drive_root_id, sheet_ids jsonb)
allowed_users(email, role['owner','member','bookkeeper'], person_id NULL)
bank_accounts(id, entity_id, iban, bank, purpose, currency, ledger_account_code)
cards(id, entity_id, bank_account_id, number, last4, holder_person_id)
people(id, name, personal_code, iban, phone, email, roles jsonb)
counterparties(id, name, type, reg_no, vat_no, personal_code, country, address, ibans text[], emails text[],
               vat_check jsonb, default_category, default_account, notes)   -- no default entity
chart_of_accounts(id, entity_id, code, name_lv, name_en, class, type, vat_code_default, active)
rules(id, key, entity_id NULL, value jsonb, source_url, source_title, verified_at, valid_from, valid_to)
policies(id, entity_id, key, value jsonb, decided_by, decided_at, decision_id)   -- the D-POL-* answers

-- capture
sources(id, kind, external_id UNIQUE, sender, original_sender, received_at, raw_meta jsonb)
documents(id, source_id, sha256 UNIQUE, mime, original_filename, new_filename, drive_file_id, drive_path,
          doc_type, supersedes_document_id NULL, extraction jsonb, extraction_model, confidence, unreadable_fields text[], flags text[])
bank_imports(id, bank_account_id, filename, sha256 UNIQUE, period_from, period_to, opening_balance, closing_balance)
bank_lines(id, bank_account_id, import_id, booking_date, value_date, txn_date, amount, dc, counterparty,
           counterparty_iban, counterparty_code, details, details_hash, card_id NULL, status,
           UNIQUE(bank_account_id, booking_date, amount, details_hash))

-- the register (see 03 for every field)
transactions(id, entity_id, entity_reason, entity_confidence, txn_no UNIQUE, type, status, period, vat_period, relates_to_year, doc_date, supply_from, supply_to,
             due_date, paid_date, booking_deadline, counterparty_id, currency, fx_rate, fx_source,
             net, vat, gross, open_amount, category, project_id, grant_id, grant_budget_line,
             vat_regime, vat_deductible_share, vat_deductible_amount, pvn1_part, pvn_itemise,
             payment_method, paid_by_person_id, purpose, participants, evidence_ok,
             income_type_code, tax_withheld, vsaoi, net_paid, notice_due_date,
             flags text[], confirmed_by, confirmed_at, posted_at, locked_at, sheet_row_ref)
transaction_lines(id, transaction_id, description, qty, unit_price, net, vat_rate, vat, account_code, project_id, grant_id)
transaction_documents(transaction_id, document_id, role)
matches(id, transaction_id, bank_line_id, amount_applied, method, confidence)

-- ledger and sub-registers (see 06)
journal_entries(id, entity_id, entry_no, date, period, transaction_id NULL, kind, reversal_of NULL, posted_by, posted_at)
journal_lines(id, entry_id, account_code, debit, credit, vat_code, project_id, grant_id, counterparty_id, memo)
periods(id, entity_id, kind['month','vat','year'], start, end, status['open','closing','closed','filed'], closed_by, closed_at)
fixed_assets(...), depreciation_runs(...), loans(...), loan_movements(...), cash_book(...),
grants(...), grant_spend(...), projects(...), donations(...), low_value_items(...), accruals(...),
inventory_counts(...), reconciliation_acts(...)

-- decisions, filings, controls, ops
decisions(id, catalog_code, entity_id, object_type, object_id, question, options jsonb, recommended jsonb,
          legal_source, answer jsonb, answered_by, answered_at, applies_to['this','counterparty','always'])
filings(id, entity_id, kind, period_start, period_end, due_date, status, export_document_id NULL, submitted_note)
control_results(id, run_at, entity_id, check_code, status['pass','warn','fail'], detail jsonb, resolved_at)
-- legal & authorities (see 12)
legal_matters(id, matter_no UNIQUE, entity_id, type, counterparty_id NULL, authority_id NULL, reference_nos text[],
              amount_principal, amount_fees, status, risk, risk_reason, escalation_required, lawyer_name NULL,
              summary, outcome jsonb, opened_at, closed_at)
legal_deadlines(id, matter_id NULL, contract_id NULL, kind, official_date, internal_date, basis, calendar_event_ids text[], done_at)
legal_plan_steps(id, matter_id, step, owner, due_date, done_at, result)
legal_documents(matter_id NULL, contract_id NULL, document_id, direction['in','out','draft','evidence'], version, sent_via, sent_at, proof_document_id NULL)
legal_links(matter_id, transaction_id NULL, contract_id NULL)
contracts(id, contract_no, entity_id, counterparty_id, type, signed_at, signers jsonb, signature_valid, start_date, end_date,
          auto_renew, notice_days, notice_deadline, total, currency, payment_schedule jsonb, penalty_terms, project_id NULL, grant_id NULL)
contract_obligations(id, contract_id, party['us','them'], description, due_date NULL, recurring, done_at)
authorities(id, name, reg_no, kind, contacts jsonb)
user_access(email, entity_id, area['books','legal'], matter_id NULL)   -- e.g. a lawyer on one matter

events_out(id, payload jsonb, sent_at, attempts, last_error)   -- see 13
jobs(...), outbox(...), ai_usage(...), audit_log(...), sheet_sync(...), schemas(id, kind, version, file_path, uploaded_at)  -- optional VID XSDs
```

## AI models and costs

- **Triage** (is it an invoice? which type?): the cheapest Claude model (Haiku class), text only.
- **Extraction:** a mid-tier model (Sonnet class) with the PDF or image input and a zod-validated schema covering all the **E** fields in `03`. Retry with the top model (Opus class) only if a validation fails or confidence < 0.85.
- **Suggestions:** rules first (counterparty history, card, IBAN, keywords), the model only for what the rules can't decide.
- **Cache by SHA-256.** Never extract twice.
- `ai_usage` per call. `AI_MONTHLY_BUDGET_EUR` (default 30): warn at 80 %; at 100 % queue non-urgent work.
- Look up the current model names and prices in Phase 0 and store them in `rules`.

## Environment

`SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, NEXT_PUBLIC_SUPABASE_ANON_KEY, ANTHROPIC_API_KEY, AI_MONTHLY_BUDGET_EUR, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN_GABRIELSK, DRIVE_ROOT_BOURZMA, DRIVE_ROOT_GES, DRIVE_ROOT_MNB, GOOGLE_LEGAL_CALENDAR_ID, EVENTS_HUB_URL, EVENTS_HUB_KEY, EVENTS_HUB_SIGNING_SECRET, WHATSAPP_TOKEN, WHATSAPP_PHONE_NUMBER_ID, WHATSAPP_VERIFY_TOKEN, WHATSAPP_APP_SECRET`
