# 12: Legal and Authorities. Contracts, authority inquiries, disputes.

The second half of the app, next to the books. It covers **Bourzma, Green Echo Studio and MNB Production** (MNB is in scope here even though its books are not kept). The goal is that every agreement, letter from VID or another authority, claim or dispute:
- has an owner and a deadline;
- has an assessment and a plan;
- has a reply drafted in Latvian with the law cited;
- shows up in Google Calendar.

Gabriels decides and sends. The app **never sends, signs or submits anything**, and never logs in to EDS, latvija.lv or a court portal.

Decided with Gabriels (19.11.2026):
- **Scope:** contracts and agreements, VID and other authorities, disputes and debt claims. Corporate and board records are out of scope for now.
- **Lawyer role:** the app drafts, Gabriels decides. A real lawyer is required only for the escalation triggers listed under "Escalation" below.
- **Alerts:** Google Calendar events.
- **Access:** works like the accounting side. Gabriels and Patriks (CEO of both companies) see everything, and other people are added later.
- **Existing material:** Gabriels gathers it into the Drive LEGAL folders; the email sweep finds the rest. From now on, everything goes to gabrielsk@.

## 1. Intake (how things get in)

| Channel | What the app does |
|---|---|
| Email to gabrielsk@ (incl. forwarded info@) | Triage also detects legal mail. Signals: senders from vid.gov.lv, ur.gov.lv, a court, a bailiff (tiesu izpildītājs), a debt collector, a municipality or a funder, and keywords such as *VID, EDS, pieprasījums, uzaicinājums, lēmums, paziņojums, akts, audits, pārbaude, līgums, vienošanās, pielikums, pretenzija, brīdinājums, prasība, tiesa, maksājuma rīkojums, izpildu, inkasso, parāds*. Matching mail goes to the **Legal inbox**, not the invoice queue. If it does both (e.g. a debt letter with an invoice), it goes to both, linked. |
| **EDS notification email** ("Jums ir jauns dokuments EDS") | The real letter sits inside EDS, which the app cannot open. The app creates a matter in status `fetch_from_eds` with a calendar event "Download the VID letter from EDS" for **today**. A deadline counts from the letter's own date, so time matters. Gabriels downloads the PDF and uploads it or drops it into Drive. |
| **Oficiālā e-adrese** notification | Same as EDS: a `fetch_from_eade` task, then manual upload. |
| **Drive inbox folder** `<entity>/LEGAL/_Ienākošie/` | Watched every 15 minutes. Anything dropped there is processed: scans, EDS PDFs, `.edoc`/`.asice` signed files. This is the main manual channel. |
| Upload button (Legal screen, phone camera) | Same processing. |
| From the books | A debt letter, a duplicate payment needing a refund (D-DUP-1), a disputed creditor claim (D-DEBT-1) or a tax risk found by controls → the app proposes opening a matter, linked to the transactions. |

**Processing:**
- **Same file pipeline as invoices** (`04` C): hash, convert, OCR, `.edoc` unzip, with the signature checked and the signers and signing time read. A file is never processed twice.
- **Extract:**
  - issuer and authority;
  - addressee entity, decided by the **name or reg. no. on the letter, never the address**;
  - reference no.;
  - date of the letter and date received;
  - what is asked or claimed;
  - amounts;
  - stated deadline;
  - legal basis quoted in the letter;
  - attachments.
- The result is linked to an existing matter (by reference no. or counterparty) or opens a new one.

## 2. Registers

### 2.1 Matters (lietas): one per case

| Field | Notes |
|---|---|
| `matter_no` | `L-BZ-2026-007`, `L-GES-…`, `L-MNB-…`, gap-free |
| `entity` | Suggested from the addressee; Gabriels confirms (D-LEG-1) |
| `type` | `authority_inquiry` (information request), `tax_audit`, `authority_decision`, `authority_other` (UR, municipality, funder), `contract_issue`, `debt_claim_against_us`, `debt_claim_by_us`, `dispute`, `refund_claim` |
| `counterparty` / `authority` | With reg. no. and the contact person from the letter |
| `reference_nos[]` | The authority's and the creditor's numbers, case numbers |
| `amount_at_stake` | Principal, claimed fees and penalties, kept separate |
| `status` | `fetch_from_eds` → `new` → `assessed` → `plan_approved` → `in_progress` → `waiting_for_them` → `resolved` → `closed` (also `escalated_to_lawyer`) |
| `risk` | `low` / `medium` / `high`, with the reason (amount, deadline, possible sanction, personal liability) |
| `escalation_required` | true if a trigger in section 5 applies |
| `deadlines[]` | Each with: kind (reply, objection, appeal, payment, submission, hearing), official date, legal basis for how it was computed, **internal date** (official minus 3 working days), and a Google Calendar event ID |
| `plan[]` | Steps with owner, due date, done/not done, and the result |
| `documents[]` | Incoming and outgoing letters, evidence and drafts, each with Drive file ID and version |
| `linked_transactions[]` | Records in the books (`03`), e.g. the invoices a claim is about, the payments made, the tax lines |
| `linked_contracts[]` | |
| `outcome` | Result, amount paid or received, the lessons learned, and whether a rule should change |
| `audit` | Who changed what, when |

### 2.2 Contracts (līgumi)

| Field | Notes |
|---|---|
| `contract_no`, entity, counterparty, type (supplier, customer, rent, grant/financing, loan, author agreement/autoratlīdzība, prize, service, NDA, other) | |
| Signing | Date, signers, signature type (`.edoc`, eParaksts, wet ink), whether the signature is valid, and whether both parties signed |
| Term | Start, end, auto-renewal, notice period, and the **last day to give notice** (computed) |
| Money | Total, payment schedule, currency, VAT, indexation, penalty clause (līgumsods), late-payment interest |
| Obligations[] | Ours and theirs, e.g.: deliver the event, submit the grant report by date X, keep records for N years, insurance, publicity requirements. Each obligation with a date becomes a deadline and a calendar event. |
| Links | Project, grant, and the transactions that pay it. The books are checked against the schedule: payments made, invoices expected and missing. |
| Annexes | Addenda (vienošanās, pielikumi) as versions; the current terms are shown merged |

### 2.3 Authority register

One row per authority contact for each entity:
- VID: assigned inspector, phone, structural unit;
- Uzņēmumu reģistrs;
- municipalities;
- grant funders.

It also holds a history of all matters with each authority.

## 3. What happens with each new matter (automatic, within minutes)

1. **Summary:** a five-line plain-English summary of what they want and by when. The original stays the authority.
2. **Deadline calculation.**
   - If the letter states a date, use it.
   - If it states a period ("within 15 days from receipt"), compute the date from the receipt rules in `rules`: when a document sent through EDS or e-adrese counts as received, working days vs calendar days, and holidays. Each rule cites the current text of the law (e.g. *likums "Par nodokļiem un nodevām"*, *Administratīvā procesa likums*, *Paziņošanas likums*, *Civilprocesa likums*), looked up and never typed from memory.
   - If the date can't be computed with certainty, show the **earliest possible date** and ask (never guess later).
   - Each deadline becomes a Google Calendar event (section 6).
3. **Assessment:**
   - what is asked;
   - which documents and figures from the books answer it;
   - risk level and why;
   - options (comply in full, comply partly with an explanation, ask for an extension, object or appeal, negotiate, pay);
   - the legal basis for each option, with links.
4. **Answer pack.** For VID requests, the app gathers what is asked straight from the books: register extracts, the invoices and contracts concerned, bank lines, the VAT registers for the period. It produces a numbered annex list (pielikumu saraksts) with each file. Nothing is created that isn't in the books; anything missing is listed as missing.
5. **Plan.** Three to eight steps with dates, ending at the internal deadline.
6. **Draft letter in Latvian:**
   - authority letter format (addressee, reference to their letter no. and date, subject, numbered answers matching their numbered questions, annex list, signatory: board member);
   - formal tone;
   - cites the law it relies on.

   For debt claims, the draft is a reply that does one of these: accepts the principal, disputes the fees with the legal reason, or proposes a payment schedule.
7. **Decision card** to Gabriels (D-LEG-2): approve the plan and the option.

## 4. Decision catalog additions (also listed in `05`)

| Code | Question | Default |
|---|---|---|
| D-LEG-1 | New legal item: which entity, which matter (new or existing), what type | The suggestion from the addressee and the reference no. |
| D-LEG-2 | Approve the assessment, option and plan | The recommended option (shown with the reason and the law) |
| D-LEG-3 | Approve the letter to send (the final text; Gabriels sends it via EDS, e-adrese, email or post and records how and when) | Approve |
| D-LEG-4 | Ask for a deadline extension? (a draft request is ready) | Yes, if the answer pack is not complete 5 working days before the deadline |
| D-LEG-5 | Bring in a lawyer (section 5) | Yes, for any trigger |
| D-LEG-6 | Close the matter: outcome, amounts, follow-up rule | Close when the outcome is recorded and no deadline is open |
| D-CON-1 | New contract: confirm parties, term, money, obligations | The extraction |
| D-CON-2 | A notice deadline is coming: renew, renegotiate or terminate? (a draft notice is ready) | Ask 30 days ahead |

## 5. Escalation: when a real lawyer is required (decided: no money threshold)

The app marks the matter `escalation_required` and shows a red banner **"Lawyer needed"** when any of these is detected:
- **Court or bailiff:**
  - a court claim (prasības pieteikums) or court summons;
  - a warning procedure or payment order (brīdinājuma kārtība / bezstrīdus piedziņa);
  - a bailiff (tiesu izpildītājs) notice or an enforcement order;
  - a VID **tax audit decision** (nodokļu audita lēmums), or a decision that can be appealed to court;
  - any appeal deadline to a court.
- **Criminal or personal liability:**
  - anything mentioning criminal proceedings (kriminālprocess), administrative offence proceedings (administratīvā pārkāpuma process) against a person, or the personal liability of a board member;
  - risk of insolvency (maksātnespēja);
  - a request to the board member personally.

**For an escalated matter, the app still:**
- computes the deadlines and creates the calendar events;
- prepares a **lawyer brief** in Latvian: the facts on a timeline, documents (numbered), amounts, the questions for the lawyer, and the deadline;
- keeps the drafts marked "DRAFT – not for sending until the lawyer reviews".

The lawyer can be added as a user with access only to that matter.

## 6. Google Calendar

- **Scope:** `calendar.events` on gabrielsk@'s calendar, in a dedicated calendar **"Juridiskie termiņi"**. Accounting filing deadlines (`07`) are added there too, so there is one place to look.
- **Events per deadline:**
  - an all-day event on the **internal date**: "⚠ [L-BZ-2026-007] VID atbilde – iekšējais termiņš";
  - an all-day event on the **official date**.

  Reminders: 7 days, 2 days and on the day.
- **Event content:** the event description holds the matter link, the one-line summary and the next plan step.
- **Sync:** when a deadline changes (e.g. an extension is granted) the events are updated, never duplicated. When the matter closes, the events stay, marked "✔ izpildīts".
- **Escalated matters** get a red event colour.
- **Patriks:** if he should see the events, the calendar is shared with him from Google (Gabriels does this; the app doesn't change sharing).

## 7. Drive structure (per entity root, incl. MNB)

```
<entity root>/LEGAL/
  _Ienākošie/                         ← drop zone, watched
  Līgumi/<counterparty>/<YYYY-MM-DD>_<type>_<counterparty>_<no>.pdf (+ .edoc kept)
  Iestādes/VID/<YYYY>/<matter_no>_<topic>/   (incoming, outgoing, annexes)
  Iestādes/<other authority>/<YYYY>/<matter_no>_<topic>/
  Strīdi un parādi/<YYYY>/<matter_no>_<counterparty>/
```
- Files are copied and renamed, never moved out of their original place. Existing files are linked, not moved.
- Outgoing letters are saved as the final PDF that Gabriels sent, plus the proof of sending (EDS receipt, email, post receipt).

## 7b. Google Sheet mirror

One sheet **"Juridiskās lietas"** in the Bourzma `LEGAL/` folder, covering all three entities: tabs *Lietas* (matters with status, risk, next deadline, next step), *Termiņi* (all open deadlines), *Līgumi* (contracts with notice dates and obligations). Same rules as the accounting Sheets: the database is the source of truth, one editable "Piezīmes (Gabriels)" column.

## 8. Controls (added to `08`)

| # | Control | Level |
|---|---|---|
| C24 | A matter with an open deadline and no approved plan within 2 working days of receipt | amber → red at 5 days before the internal date |
| C25 | `fetch_from_eds` / `fetch_from_eade` task not done by the end of the next working day | red |
| C26 | A deadline without a calendar event, or an event out of sync | red (auto-resync first) |
| C27 | A contract notice deadline within 30 days without D-CON-2 answered | amber |
| C28 | A contract payment schedule vs the books: an expected payment or invoice is missing | amber |
| C29 | Escalation required and no lawyer recorded within 3 working days | red |
| C30 | A matter whose addressee entity ≠ the confirmed entity | red |

## 9. Screen: **Legal** (sixth main screen, see `09`)

- **Top:**
  - a "Next legal step" button;
  - tiles: open matters by risk; deadlines in 14 days; waiting for EDS download; escalated; contract notices due.
- **Matters list:** sorted by internal deadline. Filters: entity (incl. MNB), type, authority, status.
- **Matter page:**
  - **left:** the timeline of letters in and out;
  - **middle:** summary, deadlines, plan checklist, linked transactions and contracts;
  - **right:** the current draft letter, with edit, "Approve (D-LEG-3)" and "Mark as sent" (how, when and proof upload).
- **Contracts list:**
  - expiring and notice dates;
  - payment schedule vs actual;
  - obligations due.
- **Authorities:** contacts and history.
- **Today screen:** legal deadlines appear in the same filing strip as tax deadlines, so there is one queue.

## 10. Start-up load

1. Create the Drive LEGAL folders for all three entities (the app does it in Phase 2L, using the Drive roots entered in Phase 0, MNB's included).
2. Gabriels gathers existing contracts and letters into `_Ienākošie/` of the right entity.
3. Index them, along with the email sweep 2024-12 → today (the same sweep as the books, with the legal keywords).
4. **Seed these known matters right away:**
   - **the VID letter with the reply deadline 20.11.2026** (Bourzma; deadline extended);
   - **the Julianus / VEF Kvartāls claim 3186319** (debt claim against us: principal 3,445.00, claimed fees 5,717.25; invoices VK-24/0441 and VK-24/0452);
   - **refund claims:** A-Sound 07/04 and GD&L V25-06-01 (paid twice), and SKR 250002 (received twice), each pending D-DUP-1;
   - **tax risk on 2025 author fees (~4,189) and prizes (1,800):** internal matter, possible correction (D-PRV-1);
   - **contracts:** Bauska BKC/2025/1-14.3.2/6, the loan from Mārtiņš Kirkils (19.12.2024).
