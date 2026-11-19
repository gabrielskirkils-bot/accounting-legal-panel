# 13: Ecosystem events. How this app talks to the personal assistant.

Gabriels runs several separate apps:
- accounting-legal-panel;
- Nexstr Academy;
- future projects;
- a **personal assistant**, connected to his personal Gmail, Calendar and other apps.

Each app owns its own data and its own Google connection. **Apps never read each other's databases or mailboxes.** They share only short **events**, sent one way to the assistant's **Hub**.

```
accounting-legal-panel ─┐
Nexstr app ─────────────┼──► POST https://<assistant>/api/events  (one key per app) ──► Hub inbox ──► assistant decides: push / digest / calendar / ignore
future apps ────────────┘
Deadlines also go to Google Calendars that each app shares with Gabriels's personal account (read-only).
```

## The event contract (the same for every app; version 1)

```json
{
  "v": 1,
  "id": "alp_2026-11-19_8f3a",           // unique per app; the Hub ignores repeats
  "source": "accounting-legal-panel",    // app name
  "entity": "BOURZMA",                   // BOURZMA | GES | MNB | null (Nexstr etc. use their own)
  "type": "deadline",                    // see list below
  "priority": "high",                    // low | normal | high | urgent
  "title": "PVN Q4 jāiesniedz",           // ≤ 80 characters, no personal codes, IBANs or card numbers
  "summary": "Bourzma VAT return due 20.01.2027; 2 decisions open.",  // ≤ 300 characters
  "due_at": "2027-01-20T23:59:00+02:00", // optional
  "action_needed": true,
  "action_url": "https://accounting-legal-panel.vercel.app/taxes/BOURZMA/2026-Q4",  // deep link; the action happens in the app
  "created_at": "2026-11-19T19:40:00+02:00",
  "supersedes": null                     // id of an earlier event this one updates or cancels
}
```

**Types:** `decision_waiting`, `deadline`, `deadline_changed`, `new_official_letter`, `escalation`, `control_red`, `filing_ready`, `reminder`, `summary_daily`, `summary_weekly`, `health_problem`, `resolved`.

## What accounting-legal-panel sends

| When | Type | Priority |
|---|---|---|
| New VID, EDS or e-adrese notice; new letter from a court, bailiff or collector | `new_official_letter` | urgent |
| Escalation trigger (`12` section 5) | `escalation` | urgent |
| A legal or tax deadline is created or changes; reminders at 7, 2 and 0 days before | `deadline` / `deadline_changed` | high (urgent at 0–2 days) |
| VAT return or paziņojums ready to approve | `filing_ready` | high |
| A red control (`08`) | `control_red` | high |
| Decision cards waiting > 24 h (one grouped event, not one per card) | `decision_waiting` | normal |
| Monday 09:00: bank upload missing | `reminder` | normal |
| Health: Gmail token, forwarding or worker down | `health_problem` | high |
| Daily 07:30 Riga: counts and next steps | `summary_daily` | low |
| Resolved or closed | `resolved` (with `supersedes`) | low |

## Rules

- **Privacy:** events carry titles, amounts and dates only. Never personal codes, IBANs, card numbers or document text. Details stay in the panel behind its login.
- **One-way and safe:**
  - The panel sends with `Authorization: Bearer <EVENTS_HUB_KEY>` and an HMAC signature header.
  - It sends through the outbox with retries, so a Hub outage never blocks the books.
  - The assistant can't change anything in the panel: it only links back via `action_url`.
- **Switch:** if `EVENTS_HUB_URL` is empty, events are only stored locally (`events_out` table), so the panel works before the assistant exists.
- **Same contract everywhere:** Nexstr and future apps copy this section into their own spec and change only `source`, `entity` and their event list.

## Data and env

- Table `events_out(id, payload jsonb, sent_at, attempts, last_error)`.
- Env: `EVENTS_HUB_URL`, `EVENTS_HUB_KEY`, `EVENTS_HUB_SIGNING_SECRET`.

## Phase

Built in Phase 2L, together with the legal deadlines. The daily summary comes in Phase 3.
