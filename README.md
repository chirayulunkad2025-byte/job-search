# jobbot — first-to-apply internship alerts on WhatsApp

Polls company careers sites (Workday, Greenhouse, Lever, Ashby, Oracle), the SimplifyJobs Summer 2027 list,
and your job-alert emails (LinkedIn, Handshake, Indeed, Jobright…) every 15 minutes on GitHub Actions.
New postings that match your filters go to your WhatsApp within minutes:

```
🆕 *2027 Summer Internship (US) - Product Management* — Zebra Technologies
📍 Lincolnshire, Illinois · posted 2026-11-14 · workday
🎯 fit 8/10 — RFID product line specs, launch support ⚠️ no sponsorship stated
🔗 https://zebra.wd501.myworkdayjobs.com/en-US/Zebra_careers/job/...
```

- **Tier A** (product management, product marketing/planning, brand, commercial, BD, market development,
  strategy, growth, pricing…) → instant message, max 8 per run.
- **Tier B** (operations, supply chain, sourcing, program/project mgmt, consulting, sales, marketing…) → one daily
  digest at 13:00 UTC (8am ET).
- Drops software/data-science/engineering, fall/spring-only, other years, MBA-only, undergrad-only, and non-US roles.
- **Weekly "opening soon" message** (Mondays): companies that haven't posted for 2027 yet but opened around this
  date last cycle. History starts in March 2026, so this gets useful from about February 2027.

## Setup (about 30 minutes)

1. **GitHub repo.** Create a **public** repository (public = unlimited free Actions minutes, so 15-minute polling
   is free). Upload this folder's contents, keeping the `.github/` folder.
   Then go to Settings → Actions → General → Workflow permissions → **Read and write permissions** → Save.
   Your secrets stay private even in a public repo; the visible parts are the code, the company list and the
   `data/` state files.

2. **WhatsApp (CallMeBot, free).** On your phone, save **+34 698 28 89 73** as a contact, then send it the WhatsApp
   message `I allow callmebot to send me messages`. It replies with your API key (if not within 2 minutes, retry
   after 24 hours).

3. **Secrets.** Go to Settings → Secrets and variables → Actions → New repository secret:
   | Secret | Value |
   |---|---|
   | `CALLMEBOT_PHONE` | your WhatsApp number with country code, e.g. `+16035551234` |
   | `CALLMEBOT_APIKEY` | the key CallMeBot sent you |
   | `ANTHROPIC_API_KEY` | from console.anthropic.com (for the fit score; optional, costs cents/day) |
   | `PROFILE` | paste `profile_template.txt` after editing it |
   | `GMAIL_ADDRESS` / `GMAIL_APP_PASSWORD` | optional, for reading job-alert emails (see step 5) |
   | `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` | optional backup channel if WhatsApp delivery fails |

4. **First runs.** Go to Actions → jobbot → Run workflow:
   - `mode = test-notify` → you should get "✅ jobbot test" on WhatsApp.
   - `mode = seed` → records everything already open, so you aren't flooded. (A normal first run also seeds.)
   After that, the 15-minute schedule takes over. Any source added later is also absorbed silently on its first run.

5. **Email alerts (optional, recommended).** Gmail needs 2-Step Verification on; then create an App Password at
   myaccount.google.com/apppasswords. Set up alerts that email you:
   - **LinkedIn** job alerts, e.g. "Product Management Intern", "Product Marketing Intern", "Commercial Strategy
     Intern", "Business Development Intern", location United States, filtered to Internship. LinkedIn emails
     are daily at best, so also turn on its mobile push notifications for the fastest signal.
   - **Handshake** saved searches with email alerts (Dartmouth-only postings show up here first).
   - **Indeed** and **Jobright** alerts for the same keywords.
   - Optionally a Gmail filter that labels these "JobAlerts"; then set `gmail_label: JobAlerts` in `config.yaml`.
   The bot reads them read-only (it never marks or deletes mail).

## Daily use

- **Add a company:** copy its careers URL into `config.yaml` (patterns are at the top of the file) and commit.
- **Tune what alerts you:** edit `tier_a_keywords`, `tier_b_keywords` and `exclude_title` in `config.yaml`.
- **Check health:** `data/last_run.json` shows per-source counts and errors. If a source fails for about 3 hours
  in a row, you get a WhatsApp warning.
- `data/new_jobs.jsonl` logs every match (with scores), so your Claude internship board can import them.

## Limits to know

- GitHub runs scheduled jobs on a best-effort basis, so runs can start a few minutes late at busy times.
- Some careers sites (iCIMS, SuccessFactors, Taleo, Phenom, Eightfold — e.g. Deere, Caterpillar, Stryker's
  marketing site) have no public JSON feed and aren't polled directly. Simplify, email alerts and
  talent-community emails cover them.
- CallMeBot is a free, unofficial service that only messages your own number. If it stops working, use the
  Telegram fallback or switch to Twilio (paid, official).
- LinkedIn, Handshake and Glassdoor are never scraped; only the alert emails they send you are read.
- Simplify only lists postings after its team adds them; direct careers-site polling is what gets you there first.

## Local testing

```
pip install -r requirements.txt pytest
pytest -q                              # offline tests
python -m jobbot.main --dry-run --only simplify
```
