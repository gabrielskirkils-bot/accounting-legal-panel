# SETUP: step by step, from zero to daily use (Mac)

Each step says **what** you do, **why** it is needed, and **when it is done**. Follow them in order and tick each box as you go. Diagrams of how the parts connect are in `docs/diagrams.md`.

**Your setup:**
- a Mac;
- mnbproduction.com is Google Workspace, and you are its admin;
- info@bourzma.com is a Google mailbox.

## Before you start: the parts in one minute

| Part | What it is | Its job here |
|---|---|---|
| **GitHub** | A safe online home for code | Keeps the code and its history; Vercel and Railway take the code from here |
| **Supabase** | The database, plus logins and file storage | **The single source of truth:** every transaction, ledger line, case and decision |
| **Vercel** | Hosting for websites | Runs the dashboard you open in the browser |
| **Railway** | Hosting for a program that runs all the time | Runs the **worker**: reads email, does OCR, calls the AI, posts entries, writes to Drive, Sheets and Calendar |
| **Google Cloud** | The "key" to your Google account | Lets the worker read gabrielsk@ mail and write to Drive, Sheets and Calendar |
| **Anthropic** | The AI | Reads invoices and letters, extracts the fields, drafts replies |
| **Claude Code** | The AI programmer on your Mac | Writes the whole app from the spec, phase by phase |

**The golden rule for keys.** Several steps give you **secret keys**. Save each one in a **password manager** (e.g. Apple Passwords or 1Password) as a secure note called `accounting-legal-panel keys`. They go only into:
1. a file `.env.local` on your Mac, which is never uploaded;
2. the settings pages of Vercel and Railway.

Never put them in chat, email or GitHub. Anyone with the Supabase secret key or the Google secret can read everything.

---

## Step 1. Install the tools on your Mac (about 30 minutes)

**Why:** Claude Code builds and tests the app on your Mac before it goes online. For that it needs these programs.

Open **Terminal** (Cmd + Space, type "Terminal", Enter). Paste each command, press Enter, and wait for it to finish.

1. **Homebrew,** the Mac's app installer that installs the others:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
   It asks for your Mac password; the typing stays invisible, which is normal. **At the end it prints two "Next steps" lines (`echo … >> ~/.zprofile` and `eval …`). Copy and run them**, or `brew` won't be found.
2. **Git, Node.js and pnpm:**
   - Git keeps versions of the code.
   - Node.js runs JavaScript, the language of the app.
   - pnpm installs the app's libraries.
   ```bash
   brew install git node pnpm
   ```
3. **Supabase CLI,** to send the database structure to Supabase:
   ```bash
   brew install supabase/tap/supabase
   ```
4. **Docker Desktop,** a "box" that runs the worker exactly as it will run on Railway. Download it from https://www.docker.com/products/docker-desktop. Choose **Apple chip** or **Intel chip** ( → About This Mac shows which). Install it, open it once, and accept the terms.
5. **Claude Code:** install it with the command on https://docs.claude.com/en/docs/claude-code (the official page has the current command). Then type `claude` in Terminal and log in with your Claude account.
6. **Check everything.** Each command should print a version number:
   ```bash
   git --version && node --version && pnpm --version && supabase --version && docker --version && claude --version
   ```
   Node must be **20 or higher**.

✅ **Done when** all six versions print without errors.

---

## Step 2. Create the online accounts (about 1 hour)

### 2.1 GitHub: the home for the code
**Why:** the code needs a safe online home with a full history, so nothing is ever lost. Vercel and Railway deploy straight from it.

1. Log in at https://github.com (create an account if you don't have one).
2. Click **+** (top right) → **New repository**:
   - **Repository name:** `accounting-legal-panel`
   - **Private** (very important: the code will contain your business set-up)
   - **Don't** tick README, .gitignore or licence. The spec already has its files.
3. Click **Create repository** and keep the page open. Copy the URL `https://github.com/<you>/accounting-legal-panel.git`. 💾 **Save**

✅ **Done when** an empty private repo exists.

### 2.2 Supabase: the database, logins and file storage
**Why:** all data lives here: transactions, the ledger, cases, decisions and users. It also sends the login links and runs the timers (every 15 minutes, every Monday, every night).

**A. Account**
1. Go to https://supabase.com → **Start your project** → **Continue with GitHub**.
2. Create an organization: name `Bourzma`, type Personal, plan **Free** (enough while building).

**B. Project**
1. Click **New project**:
   - **Name:** `accounting-legal-panel`
   - **Database password:** click **Generate a password** and copy it. 💾 **Save** it now; it is not shown again.
   - **Region:** *Central EU (Frankfurt)*. It can't be changed later. EU keeps the data under GDPR and close to Riga, so it is fast.
2. Click **Create new project** and wait 1–2 minutes.
3. The browser address now ends in `/project/abcdefghijklmnop`. That code is the **Project ref**. 💾 **Save**

**C. Keys** (gear icon **Project Settings**, bottom left)

These keys are how the app proves to Supabase that it is allowed in.
1. **Project URL:** under **Data API** (or the **Connect** button at the top), e.g. `https://abcdefghijklmnop.supabase.co` → `SUPABASE_URL`. 💾 **Save**
2. Under **API Keys**, save one pair. You'll see new-style keys, legacy keys, or both tabs:
   - **Publishable** (`sb_publishable_…`) or legacy **anon** → `NEXT_PUBLIC_SUPABASE_ANON_KEY`. 💾 **Save**. This is the "visitor" key, safe in the website, and it only works together with the access rules.
   - **Secret** (`sb_secret_…`, click *Reveal*) or legacy **service_role** → `SUPABASE_SERVICE_ROLE_KEY`. 💾 **Save**. This is the "master" key and bypasses all rules. **Only the worker and the server use it.**

**D. Timers**
1. Left menu **Database → Extensions**.
2. Search **pg_cron**, switch it on, keep the default schema, and click **Enable extension**.
3. Do the same for **pg_net**.

**Why:** pg_cron is the alarm clock (it starts the email check, the Monday reminder and the nightly checks). pg_net lets the database call out when an alarm rings.

**E. Lock the door**
1. Go to **Authentication → Sign In / Providers**:
   - **Email** stays on. That's the magic-link login: you type your email and click the link you receive.
   - Under **User Signups**, turn **off** "Allow new users to sign up" and click **Save**. Now nobody can create an account by themselves.
2. Add yourself: **Authentication → Users → Add user → Create new user**. Enter `gabrielsk@mnbproduction.com`, tick **Auto Confirm User**, and click **Create user**.

**Two things to know:**
- **Free is only for building.** Free projects pause when nothing has happened for a while, and they have no automatic backups. Upgrade to **Pro** (Organization → Billing; check the current price there) **before you rely on it**, at the end of Phase 3 at the latest.
- **The built-in login email is for testing only.** It sends a few emails an hour, and only to your Supabase team. Step 7 connects your own sender, so Patriks and the bookkeeper get their links too.

✅ **Done when** you've saved 5 things: the DB password, the project ref, the URL, the publishable/anon key and the secret/service_role key.

### 2.3 Google Cloud: the key to Gmail, Drive, Sheets and Calendar
**Why:** the worker has to read gabrielsk@ mail, put files in Drive, write the Sheets and create Calendar events. Google allows that only to a registered "app" that you approve once. Because you are the Workspace admin, you can make the app **Internal**. Then:
- only mnbproduction.com accounts can use it;
- Google doesn't review it;
- access doesn't expire after 7 days.

Do all of this **logged in as gabrielsk@mnbproduction.com** (check the avatar, top right).

**A. Project**
1. Go to https://console.cloud.google.com and accept the terms if asked.
2. Click the project picker at the top → **New project**:
   - Name: `accounting-legal-panel`
   - Organization: `mnbproduction.com`
3. Click **Create**, then select the project in the picker. Every next step must show this project at the top.

**B. Switch on the four APIs**

An "API" is a door into one Google service, and each door has to be opened separately.

Go to **☰ → APIs & Services → Library**. For each name below, search, open it and click **Enable**:
1. **Gmail API:** reading mail.
2. **Google Drive API:** filing documents.
3. **Google Sheets API:** the register sheets.
4. **Google Calendar API:** the deadline calendar.

**C. Consent screen**

This is the screen you'll see once when you approve the app. Go to **☰ → APIs & Services → OAuth consent screen**, which may show as **Google Auth Platform**, and click **Get started**:
1. **App name:** `accounting-legal-panel`. **User support email:** gabrielsk@.
2. **Audience: choose Internal.** This is the important choice.
3. **Contact email:** gabrielsk@. Agree, then click **Create**.

**D. The app's own login key**
1. Go to **Clients** (or **Credentials → + Create credentials → OAuth client ID**).
2. **Application type:** Web application. **Name:** `accounting-legal-panel`.
3. Under **Authorized redirect URIs**, click **+ Add URI** and enter:
   `http://localhost:3000/api/google/callback`

   After you approve, Google sends you back to this address on your Mac. The online address is added in Step 7.
4. Click **Create**. A window shows two values:
   - **Client ID** → `GOOGLE_CLIENT_ID`. 💾 **Save**
   - **Client secret** → `GOOGLE_CLIENT_SECRET`. 💾 **Save** (secret)

**E. Allow forwarding to reach you (Workspace admin)**

Step 4 forwards info@bourzma.com and Patriks's CityBee mail into gabrielsk@. Workspace can block automatic forwarding, so check it now:
1. Open https://admin.google.com → **Apps → Google Workspace → Gmail → End User Access**.
2. Make sure **Automatic forwarding** is allowed for the organization that owns the *sending* mailbox (bourzma.com, if it's a separate Workspace).

You don't create the final "refresh token" here. In Phase 0, Claude Code builds a command (`pnpm google:auth`) that opens Google's approval screen and gives you that token.

✅ **Done when** the 4 APIs show "Enabled", the audience is Internal, and the Client ID and secret are saved.

### 2.4 Anthropic: the AI that reads invoices and letters
**Why:** the worker sends each invoice or letter to Claude through the API, and gets back the fields (date, numbers, amounts, buyer) or a draft reply. This is separate from your Claude chat subscription: the API is paid per use.

1. Go to https://console.anthropic.com, sign up or log in, and create the organization if asked (name `Bourzma`).
2. Under **Billing**, add a card and buy a small amount of credit to start.
3. Under **API Keys → Create key**, name it `accounting-legal-panel`. **Copy it immediately, because it is shown only once.** → `ANTHROPIC_API_KEY`. 💾 **Save**
4. Under **Settings → Limits**, set a **monthly spend limit**. It is the second safety net; the app has its own budget too, with a default of 30 EUR a month and a warning at 80 %.

✅ **Done when** the key is saved and a limit is set.

### 2.5 Vercel: hosts the dashboard
**Why:** so that you and Patriks can open the dashboard from any browser or phone, without your Mac being on.

1. Go to https://vercel.com → **Sign Up** → **Continue with GitHub**. Choose the **Hobby** plan.
2. When Vercel asks for GitHub access, choose **Only select repositories** → `accounting-legal-panel`.
3. Stop there. You connect the project in Step 7, after the app works on your Mac.

✅ **Done when** you can log in to Vercel and it can see the repo.

### 2.6 Railway: runs the worker
**Why:** the worker must run all the time. It checks mail every 15 minutes, converts files, runs OCR, calls the AI, writes to Drive, Sheets and Calendar, and runs the nightly checks. Vercel can't do this: it stops long jobs after a short time and has no OCR tools. Railway runs the worker's Docker box non-stop.

1. Go to https://railway.app → **Login** → **GitHub**.
2. Choose the **Hobby** plan (check the current price on the site). On the free trial the worker would stop when the trial ends.
3. Stop there. The worker is deployed in Step 7.

✅ **Done when** you have a Railway account on a paid plan.

### 2.7 WhatsApp
Skip it. It comes in Phase 5.

---

## Step 3. Gather the information (one evening)

**Why:** the app must start with the real facts: which folders, which people, which cards, what the books looked like at 01.01.2025. It must never guess them. Keep everything in one private folder on your Mac (not inside the repo). Claude Code asks for it in Phase 0.

1. **Drive folder IDs.** A folder's ID is the part of its web address after `/folders/`.
   - Bourzma: open `BOURZMA - BIEDRĪBA/ACCOUNTING/FINANCES` and copy the ID → `DRIVE_ROOT_BOURZMA`.
   - Green Echo: create `GREEN ECHO STUDIO/ACCOUNTING/FINANCES` and copy its ID → `DRIVE_ROOT_GES`.
   - MNB Production (legal only): pick or create its root folder and copy the ID → `DRIVE_ROOT_MNB`.
   - Make sure gabrielsk@ is the owner or an editor of all three.
2. **Legal calendar.** In Google Calendar (as gabrielsk@):
   - Click **+** next to "Other calendars" → **Create new calendar**, name it **`Juridiskie termiņi`**, and click **Create**.
   - Open its **Settings → Integrate calendar** and copy the **Calendar ID** → `GOOGLE_LEGAL_CALENDAR_ID`.
   - To let Patriks see the deadlines: **Share with specific people** → Patriks, with "See all event details".
3. **People:** name, personal code, IBAN, phone and email for Gabriels, Patriks, Kristaps, Terēze, Mārtiņš Kirkils and Kaspars Mārtiņkrists. **Why:** payments to and from people (loans, author fees, card holders) are booked by person.
4. **Full card numbers** for Bourzma …1187, …7665 and …7441 (from the statements). **Why:** the card printed on a receipt decides which company it belongs to.
5. **Bank exports.** Download the **original CSV files** from internet banking. Don't open them in Excel and re-save, because that changes the format.
   - Swedbank, both Bourzma accounts, 01.01.2025 → today.
   - Luminor, both GES accounts, 01.01.2025 (or GES's start) → today.
6. **Opening figures:**
   - Bourzma's **2024 annual report** (PDF);
   - GES's last annual report if there is one, and its start date.

   **Why:** the closing figures of 2024 are the opening figures of 2025.
7. **Agreements:** the loan agreement with Mārtiņš Kirkils (19.12.2024).
8. **Bookkeeper:** name, email and agreement date, when appointed. This can wait.
9. **Test documents:** copy the 14 files listed in `docs/11-phases-and-tests.md` (G1–G14) from Drive into `fixtures/golden/` in the repo (after Step 5). **Why:** they are the "exam" the app must pass in Phase 1. Each one is a real case that once caused trouble.

✅ **Done when** everything is in one private folder.

---

## Step 4. Send all mail into one inbox (about 20 minutes)

**Why:** the app reads **one** mailbox, gabrielsk@mnbproduction.com. Everything else is forwarded there automatically, so nothing is missed.

**A. info@bourzma.com → gabrielsk@**
1. Log in to **info@bourzma.com** in Gmail. Go to ⚙️ **Settings → See all settings → Forwarding and POP/IMAP**.
2. Click **Add a forwarding address** and enter `gabrielsk@mnbproduction.com` → **Next → Proceed**.
3. Google sends a **confirmation code to gabrielsk@**. Open it there and copy the code (or click the link).
4. Back in info@: paste the code → **Verify**.
5. Choose **Forward a copy of incoming mail to gabrielsk@…** and **keep info@'s copy in the Inbox**. Click **Save Changes** at the bottom.

**Why keep the copy:** info@ stays complete on its own, and the app never deletes anything.

**B. Patriks's CityBee mail (spended0@gmail.com) → gabrielsk@**

Patriks does this in his Gmail (only CityBee mail, not his private mail):
1. Do steps 1–4 of part A in spended0@, so gabrielsk@ is added and verified as a forwarding address. **Don't** switch on "Forward a copy of all incoming mail".
2. In the search bar, type `from:(citybee.lv OR citybee.lt)` → click the ⚙️ filter icon on the right → **Create filter**.
3. Tick **Forward it to:** gabrielsk@mnbproduction.com → **Create filter**.

**C. Old mail (forwarding only covers new mail)**

Export the history once with **Google Takeout**:
1. Go to https://takeout.google.com (as info@). Click **Deselect all**, tick only **Mail**, then **Next step** → **Export once**, .zip → **Create export**.
2. Download the file when Google emails you; this can take hours. Patriks does the same for spended0@. He can limit it to CityBee by first giving those mails a label, then choosing only that label under "All Mail data included".

Keep the files for Step 8. The app reads them once and skips duplicates.

✅ **Done when** a test email sent to info@ shows up in gabrielsk@ within a minute.

---

## Step 5. Put the spec into GitHub (10 minutes)

**Why:** Claude Code builds inside this folder. CLAUDE.md is its instruction book; it reads it at every start.

In Terminal:
```bash
cd ~/Documents
mkdir accounting-legal-panel && cd accounting-legal-panel
# Unzip accounting-legal-panel-spec.zip INTO this folder (double-click the zip, then move its
# contents here). CLAUDE.md, SETUP.md, docs/, fixtures/ must sit directly in this folder.
ls          # should show: CLAUDE.md  SETUP.md  docs  fixtures  reports
git init
git add .
git commit -m "Spec"
git branch -M main
git remote add origin https://github.com/<you>/accounting-legal-panel.git
git push -u origin main
```

If `git push` asks for a password, GitHub needs a token instead. The easiest fix is `brew install gh` → `gh auth login` (choose GitHub.com, HTTPS, log in with the browser), then run `git push -u origin main` again.

✅ **Done when** the files are visible in the GitHub repo page.

---

## Step 6. Build it with Claude Code, one phase at a time

**Why phases:** each phase adds one working piece and is tested on your real cases before the next one starts. If something is wrong, it is caught early and costs little to fix.

**How:**
1. In Terminal: `cd ~/Documents/accounting-legal-panel` → `claude`.
2. Give the phase prompt:
   > "Read CLAUDE.md and docs/11-phases-and-tests.md. Start Phase N. Use plan mode first, show me the plan, then build it. Tests first. At the end write reports/PHASE_N.md and tell me exactly what to test."
3. Read the plan. Say "go" if it matches the spec, or say what's different.
4. Answer its questions. It may never invent a policy, so it asks.
5. At the end, test what `reports/PHASE_N.md` says. Then save the work:
   `git add . && git commit -m "Phase N" && git push`.

| Phase | What gets built | What you do | Done when |
|---|---|---|---|
| **0: Setup** | Skeleton, database tables, login, settings, the law lookups | Paste the keys from Step 2 into `.env.local` when asked. Run `pnpm google:auth`, log in as gabrielsk@ and click Allow; it prints the refresh token. Give the Step 3 data. Answer the policy wizard (D-POL-1…10); the recommended answers are fine. | `pnpm doctor` is all green. You can log in on `http://localhost:3000`. Both companies show the right numbers, accounts and cards. |
| **1: Core flow** | Mail → reading → your confirmation → Drive + register + ledger + Sheet | Send a test invoice to info@. Confirm it on the Today screen. | The file is in the right Drive folder with the right name, the row is in the Sheet, the journal entry exists, and the tests G1–G14 pass. |
| **2L: Legal core** (alongside 2) | Legal inbox, cases, contracts, deadlines, Calendar | Put the existing contracts and letters into each company's `LEGAL/_Ienākošie/`. Confirm the pre-loaded cases. | Tests L1–L8 pass. All open deadlines are in "Juridiskie termiņi". |
| **2: Bank** | Bank upload, matching, payables and receivables | Upload the CSVs from Step 3. | Tests B1–B6 pass. Every bank line sits in exactly one status tile. |
| **3: Ledger + VAT** | Full books, month close, VAT, controls, loading 2025 | Answer the migration cards. Approve the Bourzma deduction rate (D-VAT-3). | Trial balance = 0. One real 2025 quarter's PVN matches a hand check. |
| **4: Year end + reports** | Year-end close, annual report drafts, filing calendar | Review the 2025 annual-report draft. | The draft ties to the books, and the comparatives match the 2024 report. |
| **5: WhatsApp + hardening** | WhatsApp receipts, backups, health screen | Choose WhatsApp option A, B or C. | The nightly backup runs and the failed-jobs screen works. |

**When something is wrong:** don't fix it by hand. Tell Claude Code exactly what you expected, with the real document as the example ("G7 should match the 750.00 version, it matched 749.99").

---

## Step 7. Put it online (about 1 hour, after Phase 1 works on your Mac)

**Why:** until now everything ran on your Mac. Now the dashboard goes to Vercel and the worker goes to Railway, so they work while your Mac is closed.

1. **Database structure → Supabase.** In Terminal, in the project folder:
   ```bash
   supabase login
   supabase link --project-ref <your project ref>
   supabase db push
   ```
   **Why:** this creates the tables in the online database, exactly as they are on your Mac.
2. **Dashboard → Vercel:**
   - **Add New → Project** → import `accounting-legal-panel`.
   - **Root Directory:** the web app folder (`web`, as named in `reports/PHASE_0.md`).
   - **Environment Variables:** add every line of `.env.example` with the real values from your password note.
   - **Deploy.** Copy the address, e.g. `https://accounting-legal-panel.vercel.app`.
3. **Worker → Railway:**
   - **New Project → Deploy from GitHub repo** → `accounting-legal-panel`.
   - In the service **Settings**, set the root directory or Dockerfile path to the worker (as named in `reports/PHASE_0.md`).
   - **Variables:** add the same values.
   - **Deploy.** It needs no public address, because nobody visits it; it only works.
4. **Login on the internet:**
   - In Supabase → **Authentication → URL Configuration**: set **Site URL** to the Vercel address, and add `<Vercel address>/**` under **Redirect URLs**.
   - **Users:** add Patriks (and later the bookkeeper) under **Authentication → Users → Add user** (Auto Confirm). Then add them to the app's allowed list in **Settings → Users**.
   - **Email sender:** Supabase → **Authentication → Emails → SMTP Settings** → enable custom SMTP:
     - host `smtp.gmail.com`, port `465`;
     - user gabrielsk@ (or a no-reply mailbox), with a Google **app password** (Google Account → Security → App passwords);
     - sender name `accounting-legal-panel`.

     **Why:** otherwise login links only reach you.
5. **Google:** in Google Cloud → **Clients** → your client, add `https://<Vercel address>/api/google/callback` under **Authorized redirect URIs** → **Save**.
6. **Check:** open the Vercel address, log in, and go to **Settings → Health**. Gmail, Drive, Sheets, Calendar, Anthropic and the worker heartbeat must all be **green**.
7. **Supabase Pro:** upgrade now, or at the latest after Phase 3, for automatic backups and no pausing.

✅ **Done when** Health is all green, and a test mail to info@ appears on the Today screen within 15 minutes without your Mac.

---

## Step 8. Load the history (one or two evenings)

**Why:** the app must know everything that happened before it existed, so the 2025 books can be closed and 2026 starts from the right figures.

The order is **A → C → B** (`docs/10-migration.md`):
1. **A, the 2025 work:** the v9 workbook in `fixtures/` is imported. Answer the cards it creates; your earlier decisions (Pittore, Euronics, Dokobit, Burka …) are already filled in.
2. **C, opening balances:** upload the 2024 annual report, check the proposed opening balance and approve it.
3. **B, back-fill:**
   - upload all bank CSVs;
   - upload the Takeout files from Step 4 on the **Upload** page;
   - the Gmail sweep of gabrielsk@ runs by itself.

   Then clear the decision queue. **"Confirm all ≥ 95 %"** handles routine receipts in one go.
4. **Close 2025:**
   - month by month (D-CLOSE-1);
   - then the year-end cards (D-YEAR-1);
   - then review the annual-report draft with the bookkeeper (D-YEAR-2).

---

## Step 9. Your routine after that

| When | What | Why | Time |
|---|---|---|---|
| **Daily (or every 2–3 days)** | **Today** → clear the decision cards; Enter = recommended | Nothing is filed or booked until you confirm | 2–5 min |
| **Monday** | Upload last week's bank CSVs (one per account); the reminder says which | The bank is how the app knows what was paid | 5 min |
| **By the 10th** | Close last month (D-CLOSE-1) when the checks are green | Locks the month so nothing changes it by accident | 5 min |
| **GES: by the 20th, monthly** | **Taxes** → approve PVN (D-VAT-2) → download → file in EDS → type the EDS number back | GES files VAT monthly | 15 min |
| **Bourzma: by the 20th after each quarter** | Same | Bourzma files quarterly | 15 min |
| **When a VID/EDS or e-adrese notice arrives** | Download the letter the same day and drop it into `LEGAL/_Ienākošie/` | The app can't open EDS; deadlines run from the letter | 2 min |
| **When a legal card appears** | Approve the plan (D-LEG-2), then the letter (D-LEG-3); send it yourself → "Mark as sent" | You decide and send; the app tracks it | 10–30 min |
| **Payments to private persons** | Approve the paziņojums (D-PRV-2) | Legal duty to report payments to people | 5 min |
| **January–March** | Year-end cards, annual reports, signing in EDS (board + bookkeeper) | Annual reports are due in spring | 1–2 evenings |

## If something goes wrong

- **No new invoices appear:** check **Settings → Health**. Usually either the forwarding stopped (Step 4) or Google access needs renewing (`pnpm google:auth`).
- **A bank upload is refused:** read the message. It's either the other company's file, or the balances don't tie out because a line is missing or the period is wrong.
- **A red check:** click it to see the records behind it. Fix the cause, never the number.
- **A wrong decision:** in **Register**, open the record → **Reverse**, then answer again. Nothing is ever deleted.
