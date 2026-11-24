# YardContract

A recurring lawn-care service agreement builder for independent lawn care
businesses (solo operators and small crews) running roughly 10–50 recurring
properties — the ones still pricing from gut feel, verbal agreements, and
spreadsheets. Tagline: **"Price smarter. Lock in recurring lawn care
revenue."**

YardContract is deliberately **not** a Jobber or LMN replacement: no
scheduling, no invoicing, no CRM. The market's full business platforms
(e.g. LMN at $297–$648/month) are far more than a small operator needs just
to price a route and get agreements signed. YardContract is the simple
standalone version — profitable pricing and professional agreements only.

**Founding-member pricing** for the first 25 customers, locked in while the
subscription stays active: **$19/month**, or **$129/year founding annual**
($10.75/month — a $99 saving vs paying monthly, and a better fit for a
seasonal business). Standard prices later: **$29/month** or **$199/year**.
The builder and a watermarked agreement preview are free, and the first
clean PDF agreement is free; Pro unlocks unlimited PDFs, saved agreements,
the client dashboard, the renewal center, and backup export.

Plain static HTML/CSS/JS — no build step, no backend.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page: promise, how it works, free/Pro features, a static sample agreement, founding pricing (monthly + annual), privacy + billing/cancellation notes. |
| `app.html` | The tool: agreement builder (lot tiers, surcharges, real season dates, billing modes, seasonal services with private costs), watermarked preview, PDF agreement (jsPDF via CDN), client dashboard with statuses and revenue totals, renewal center, backup export/import + CSV export. |
| `README.md` | This file. |

## Before launch

- In `index.html`, replace the `STRIPE_LINK` placeholder hrefs (hero +
  pricing monthly buttons) with the real Stripe subscription payment link
  for YardContract ($19/month founding price), and the `STRIPE_LINK_ANNUAL`
  placeholder hrefs (hero + pricing annual buttons) with the $129/year
  founding-annual link. In `app.html`, replace the `STRIPE_LINK` and
  `STRIPE_LINK_ANNUAL` constants at the top of the script (they feed the
  Pro gate subscribe buttons). The app's unlock flow is the same
  honor-system flag regardless of which plan the customer bought.
- Contact email in the footer is `chrisla122787@gmail.com`.

## Free vs Pro

**Free (no payment, no account):**

- Full builder — every input, live agreement math, calendar visit counts,
  all three billing modes, whole-contract profit view, low-margin warning.
- Watermarked agreement preview rendered on screen (diagonal
  "PREVIEW — YardContract" watermark), showing company name, client, scope,
  schedule, prices, and the billing plan.
- **First clean PDF agreement free** — one complete branded PDF, no
  watermark, counted in `localStorage` (`yardcontract.freepdf.v1`). The
  counter is only incremented *after* a PDF is successfully generated, so a
  failed PDF-library load never burns the free one.

**Pro (subscription, monthly or annual):**

- Unlimited clean branded PDF agreements: company logo + contact info,
  client and property, services + frequency + real season dates, per-visit
  pricing, season total, the billing schedule, scope included & excluded,
  weather-delay terms, payment terms, cancellation notice period, signature
  blocks for **both** parties (printed name, signature, date), and a
  discreet footer: *"Template only — not legal advice. You are responsible
  for compliance with local laws."*
- Saved agreements + client dashboard (statuses, active count, monthly
  recurring revenue, season revenue, duplicate).
- **Renewal center** — renew any saved agreement for next season in one
  click (see below).
- Backup export (JSON) / import-restore, and CSV export of agreements.

**The client PDF and preview never show internal numbers.** Crew cost,
hours, fuel/equipment/overhead, per-service costs, profit, and margin appear
only in the private "Your numbers" panel — the agreement carries services,
schedule, prices, and the billing plan only.

## ⚠️ MVP access note (read before launch)

Pro unlocking is **client-side only** for launch validation:

- The "I already subscribed — unlock Pro" button in `app.html` simply sets a
  `localStorage` flag (`yardcontract.pro.v1 = "1"`) in the visitor's own
  browser. There is **no real payment verification** — no server checks the
  Stripe subscription, and anyone who finds the button can unlock Pro without
  paying. It works the same whether the customer bought the monthly or the
  annual plan.
- This is a deliberate MVP trade-off to validate demand cheaply on a $0
  static stack (the landing page says so in plain words: unlocking is on the
  honor system during opening validation).
- Before scaling past validation, replace this with a real entitlement check
  (e.g. a small serverless function that verifies the Stripe subscription by
  email before issuing an unlock token).

## The agreement formula

All math is in `calculate()` in `app.html`:

```
tierForSqft(sqft)   sqft < 5,000 → Small ($45) · < 10,000 → Medium ($60)
                    < 20,000 → Large ($80) · 20,000+ → XL ($110)
                    (defaults; the base price field itself is editable)

perVisitPrice       = basePrice + Σ (checked surcharges)
coreSeasonTotal     = perVisitPrice × visitsPerSeason
seasonalTotal       = Σ (service price × season count)
seasonTotal         = coreSeasonTotal + seasonalTotal
monthlyEquivalent   = seasonTotal ÷ seasonMonths     (in-season average)
annualTotal         = seasonTotal                    (one season per year)

costPerVisit        = crewRate × hoursPerVisit + expensesPerVisit
                      + fuelPerVisit + equipmentPerVisit + overheadPerVisit
profitPerVisit      = perVisitPrice − costPerVisit
marginPct           = profitPerVisit ÷ perVisitPrice × 100   ← share of PRICE

contractRevenue     = seasonTotal
contractCost        = visits × costPerVisit + Σ (service cost × season count)
contractProfit      = contractRevenue − contractCost
contractMarginPct   = contractProfit ÷ contractRevenue × 100
```

### Visit counting — calendar convention

Visits are counted on the **real calendar** between the season start and
end dates (this replaced the old months × 52 ÷ 12 estimate):

- The first visit falls **on the season start date**.
- **Weekly** = every 7 days from the start; **biweekly** = every 14 days;
  **monthly** = the same day of each following month, clamped to that
  month's last day (Jan 31 → Feb 28 → Mar 31).
- A visit counts when it lands on or before the season end date.
- Season months = the calendar months the season touches, inclusive
  (Apr 1 – Oct 31 = 7; a season may wrap the year end, e.g. Nov 1, 2027 –
  Apr 30, 2028 = 6).

Worked counts, Apr 1 – Oct 31, 2026 (213 days): weekly **31**, biweekly
**16**, monthly **7**. A **visits override** field lets the operator replace
the calendar count (weather make-ups, mid-season start); when set, it wins
everywhere and the UI shows both the override and the calendar count.

Agreements saved by the older month-picker version (which stored
`seasonStart`/`seasonEnd` month numbers) still work: their months are
converted to dates (1st of the start month → last day of the end month,
wrapping the year) for display, and totals are computed with the same
calendar math.

### Billing modes — how the client pays

Service frequency (how often you visit) and billing mode (how the client
pays) are separate choices:

- **Per visit** — the client pays the per-visit price for each completed
  visit.
- **Seasonal flat rate** — the season total split into equal monthly
  payments across the season months.
- **Annual flat rate** — the season total split into 12 equal monthly
  payments, year-round.

Flat-rate installments are computed in cents: each installment is the
season total ÷ number of payments, rounded to the nearest cent, and the
**final installment absorbs the remainder** so the payments sum *exactly*
to the season total. Example ($1,285.00 season): seasonal = 6 × $183.57 +
a final payment of **$183.58**; annual = 11 × $107.08 + a final payment of
**$107.12**. When the final payment differs, it is shown on the agreement.

Dashboard "monthly recurring" follows each agreement's own billing mode:
annual flat-rate agreements contribute their installment every month;
per-visit and seasonal agreements contribute their monthly equivalent
(season total ÷ season months).

### Tier-follow rule

Typing a lot size selects the matching tier, and the base price follows to
that tier's default **only while the base price still equals the previous
tier's default** — a hand-set price is never overwritten. Picking a tier
directly always resets the base price to that tier's default.

### Why margin is profit ÷ price

A profit margin is a share of the final price. A $35.50 profit on a $70.00
visit is a 50.7% margin — even though it's a 102.9% markup on the $34.50
cost. Pricing and warnings use the true share, so the margin shown is the
margin kept.

### Worked example — the app defaults

8,500 sq ft Medium lot, base $60, slope surcharge $10, biweekly,
Apr 1 – Oct 31, 2026, fertilization 3 × $55 (cost $0); crew $38/hr ×
0.75 hr + $6 other cost, fuel/equipment/overhead $0:

- Per-visit price: $60.00 + $10.00 = **$70.00**
- Visits per season (calendar): **16** · Core season total: **$1,120.00**
- Seasonal services: **$165.00** · Season total: **$1,285.00**
- Monthly equivalent: $1,285.00 ÷ 7 = **$183.57** (183.5714…, rounded)
- Billing: per visit $70.00 · seasonal flat 7 × $183.57 (final $183.58) ·
  annual flat 12 × $107.08 (final $107.12)
- Cost per visit: $38.00 × 0.75 + $6.00 = **$34.50**
- Profit per visit: **$35.50** · Margin: 35.50 ÷ 70.00 = **50.7%**
- Whole contract: revenue **$1,285.00**, cost 16 × $34.50 = **$552.00**,
  profit **$733.00**, contract margin **57.0%**

Warning: per-visit margin under 10% shows a plain-language caution.

## Renewal center

The dashboard's Renewal Center lists every saved agreement with a
**"Renew for next season"** action. Renewing:

1. Deep-copies the agreement (the original is never modified — saved
   agreements are snapshots and stay preserved).
2. Shifts the season start/end dates **one year forward** (same month/day,
   clamped for Feb 29).
3. Applies the operator's **price adjustment %** (default +3%, editable) to
   every client-facing service price: base mowing price, surcharge amounts,
   and every seasonal service price — each rounded to the cent. Costs are
   *not* adjusted.
4. Saves the copy as a fresh **Draft** named `"<original> (renewed)"`.

## Data & privacy

Everything lives only in the visitor's own browser (`localStorage`):

| Key | Contents |
|---|---|
| `yardcontract.agreements.v1` | Saved agreements (typed inputs, name, status, date). Saved agreements are snapshots — dashboard totals are recomputed from the saved inputs. |
| `yardcontract.settings.v1` | Company name/phone/email + logo (data URL). |
| `yardcontract.pro.v1` | Pro unlock flag (see MVP note above). |
| `yardcontract.freepdf.v1` | Count of free PDFs used (0 or 1). |

Nothing is sent to a server. Records live only in that browser on that
device — they do **not** sync to a phone or another computer, and there is
no account holding a copy. The landing page and the app both say so plainly
and tell users: "export a backup regularly — that backup file is your
protection." Backup export downloads a JSON file with agreements +
settings; import restores it (files from other apps and malformed files are
rejected with nothing changed; backups saved by the older month-picker
version import fine). CSV export lists agreements with their computed
prices and billing schedule.

Agreement statuses: Draft, Sent, Active, Paused, Ended. Dashboard totals
count **Active** agreements only: active count, monthly recurring revenue
(billing-aware, see above), and season revenue (sum of season totals).

## Deploy

Any static host works — copy the three files (or at least the two HTML
files) to the host as-is:

- **Netlify / Vercel:** drag-and-drop the folder, or connect the repo; no
  build command, publish directory = this folder.
- **Cloudflare Pages:** same — framework preset "None", no build command.
- **GitHub Pages:** fine for demos/preview only — GitHub Pages terms
  prohibit primarily commercial SaaS use, so don't run the paid product
  from there.

The only external request the app makes is the jsPDF CDN script in
`app.html` (cdnjs). If it fails to load, everything except the PDF button
still works and the user gets a clear message.
