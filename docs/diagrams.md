# Diagrams: how accounting-legal-panel is built and how it works

The diagrams are in Mermaid. GitHub shows them as pictures, and Claude Code can read them.

## 1. The parts and who talks to whom

```mermaid
flowchart TB
  subgraph You["You"]
    Browser["Browser / phone<br/>(dashboard)"]
    Mac["Your Mac<br/>Claude Code builds here"]
  end

  subgraph Sources["Where things come from"]
    Gmail["Gmail gabrielsk@<br/>(+ forwarded info@, CityBee)"]
    DropDrive["Drive _Ienākošie<br/>(legal drop folder)"]
    BankCSV["Bank CSV upload<br/>(Swedbank, Luminor)"]
    WA["WhatsApp<br/>(Phase 5)"]
  end

  subgraph Cloud["Runs online"]
    Vercel["Vercel<br/>dashboard website"]
    Supabase[("Supabase<br/>database = source of truth<br/>+ login + file storage + timers")]
    Worker["Railway worker<br/>reads, converts, OCR,<br/>posts, checks, writes out"]
  end

  subgraph Out["Where results go"]
    Drive["Google Drive<br/>filed documents"]
    Sheets["Google Sheets<br/>registers (mirror)"]
    Cal["Google Calendar<br/>Juridiskie termiņi"]
    Files["Downloads for you<br/>PVN export, drafts, reports"]
  end

  Claude["Anthropic API<br/>(Claude reads & drafts)"]
  EDS["VID EDS / latvija.lv<br/>(only YOU log in)"]
  GitHub["GitHub<br/>code"]

  Mac -- push code --> GitHub
  GitHub -- deploys --> Vercel
  GitHub -- deploys --> Worker

  Browser <--> Vercel
  Vercel <--> Supabase
  Worker <--> Supabase

  Gmail -- every 15 min --> Worker
  DropDrive -- every 15 min --> Worker
  BankCSV -- you upload --> Vercel
  WA -. webhook .-> Vercel

  Worker <--> Claude
  Worker --> Drive
  Worker --> Sheets
  Worker --> Cal
  Supabase --> Files
  Files -- you file it --> EDS
```

**In words:**
- **Supabase is the heart.** Every fact lives there.
- **The website (Vercel)** shows and edits that data.
- **The worker (Railway)** does the heavy lifting. It reads mail and files, asks Claude to read documents, writes the books, and copies the results out to Drive, Sheets and Calendar.
- **Nothing goes to VID by itself.** You download the export and file it in EDS.

## 2. An invoice from arrival to the books

```mermaid
sequenceDiagram
  autonumber
  participant S as Supplier
  participant G as Gmail gabrielsk@
  participant W as Worker
  participant AI as Claude (API)
  participant DB as Supabase
  participant U as You (Today screen)
  participant D as Drive / Sheets

  S->>G: Invoice email (to info@, forwarded)
  W->>G: Check every 15 min
  W->>W: Is it an invoice? Convert, OCR
  W->>AI: Read the document
  AI-->>W: Fields: date, no., amounts, buyer…
  W->>DB: Create ONE transaction record<br/>+ suggestion (company, category, VAT, match)
  DB-->>U: Decision card appears
  U->>DB: Enter = confirm (or pick another option)
  DB->>DB: In one go: register + ledger entry<br/>+ VAT/payables/bank + checks
  W->>D: File renamed PDF in Drive, write Sheet row
  Note over U,D: Nothing is filed or booked before you confirm
```

## 3. Bank statement → matching

```mermaid
flowchart TD
  A["Monday reminder"] --> B["You upload the bank CSV"]
  B --> C{"IBAN belongs to<br/>this company?"}
  C -- no --> X["Refused: wrong company"]
  C -- yes --> D{"Opening = last closing?<br/>Lines add up?"}
  D -- no --> Y["Refused: shows the gap"]
  D -- yes --> E["Lines saved"]
  E --> F{"Automatic type?"}
  F -- "fee / tax / own transfer / loan / cash" --> G["Booked by rule"]
  F -- other --> H["Match: reference → amount+date → card day-sum → partial"]
  H -- found --> I["Linked: invoice now PAID"]
  H -- "double payment" --> J["Card D-DUP-1"]
  H -- "no document" --> K["Card D-DOC-2 / D-INC-1<br/>+ draft request in Latvian"]
```

## 4. A letter from VID (legal part)

```mermaid
flowchart TD
  A["VID email: 'Jums ir jauns dokuments EDS'"] --> B["Worker creates task:<br/>download from EDS TODAY<br/>(Calendar event)"]
  B --> C["You download the PDF and drop it into<br/>LEGAL/_Ienākošie"]
  C --> D["Worker + Claude read it"]
  D --> E["Case L-BZ-2026-xxx:<br/>summary, deadline (from the law), risk"]
  E --> F["Calendar: internal date (−3 working days)<br/>+ official date"]
  E --> G{"Court, bailiff, audit decision,<br/>personal liability?"}
  G -- yes --> H["RED: lawyer needed<br/>+ lawyer brief"]
  G -- no --> I["Answer pack from the books<br/>+ plan + draft letter (LV)"]
  I --> J["You approve plan (D-LEG-2)"]
  J --> K["You approve letter (D-LEG-3)"]
  K --> L["You send via EDS → 'Mark as sent'"]
  L --> M["Waiting for VID → next letter joins same case"]
```

## 5. The life of one transaction record

```mermaid
stateDiagram-v2
  [*] --> captured: document or bank line arrives
  captured --> awaiting_decision: something needs you
  captured --> confirmed: rule-only items (fees)
  awaiting_decision --> confirmed: you answer the card
  confirmed --> posted: ledger + registers + Drive + Sheet
  posted --> reconciled: payment matched, open amount 0
  reconciled --> locked: month closed / VAT filed
  posted --> posted: correction = reversal + new entry
  locked --> [*]
```

## 6. Your month at a glance

```mermaid
flowchart LR
  D["Daily: clear Today cards<br/>2–5 min"] --> M["Monday: upload bank CSVs<br/>5 min"]
  M --> C["By the 10th: close last month<br/>5 min"]
  C --> V["By the 20th: approve PVN → file in EDS<br/>(GES monthly, Bourzma quarterly)"]
  V --> D
  L["Any time: VID/legal letter →<br/>drop in _Ienākošie → approve plan & letter"] -.-> D
```
