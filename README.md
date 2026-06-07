# 💰 MoneyTrack

A private money tracker that works on **phone, tablet, and computer** and installs to your home
screen like a real app. It works fully offline, and can **optionally sync across all your devices**
when you sign in (see *Cloud sync setup*). Without sign-in, data stays only on the device.

**Responsive:** mobile uses a bottom tab bar; desktop/laptop uses a left sidebar with a wider
two-column dashboard; tablet sits cleanly in between.

## What it does
- **Add** daily expenses with your own custom categories (emoji + color).
- **History** — browse spending by month and day.
- **Report** — pick any period (This month, Last month, Last 30 / 90 / 120 days, This year, All
  time, or a custom **From–To**) and see total spent, daily average, top category, a by-category
  breakdown (doughnut + bars), and a spending-over-time chart (daily for short ranges, monthly
  for long ones). **Export the report to CSV** to open in Excel/Google Sheets.
- **Budget** — set monthly limits per category with OVER / NEAR / OK warnings, plus a built-in
  money advisor (works offline; optionally smarter with your own OpenAI key).
- **Plan** — your money-management engine:
  - **Monthly money plan** — converts your paycheck (weekly / every 2 weeks / twice a month /
    monthly) to a monthly figure, splits your spending into **Fixed bills** vs **Flexible**, and
    shows your **spare money**.
  - **🎯 Priorities** — one weighted list for everything you're working toward: a debt (student
    loan), a recurring send (parents), or a purchase (laptop, phone). Fixed-dollar priorities
    (e.g. a loan minimum) come out first; the rest of your spare money is split by **percentage
    weight**, with a monthly amount and an ETA for each. Log payments/sends to track progress.
  - **🔍 Spending patterns** — this week vs last week, 30-day daily average, and which categories
    are trending up or down.
  - **💡 Ways to save** — rule-based tips that flag categories creeping up and suggest cheaper
    swaps (incl. a budget meal rotation). Tap **Get specific swaps (AI)** for advice tailored to
    exactly what you bought (uses the optional OpenAI key; works offline without it).
- **Settings** — paycheck amount + frequency, savings, emergency-fund size, currency, category
  **kinds** (Fixed bill / Variable bill / Flexible) with optional expected amounts, and
  **Export / Import** a JSON backup.

## How the plan is calculated
```
Monthly income   = paycheck × (paychecks per year ÷ 12)   e.g. biweekly = ×26÷12
Spare money      = Monthly income − Typical spending (Fixed bills + Flexible)
Fixed priorities = their $/month, taken off the top first
Remaining        = Spare money − Fixed priorities
% priorities     = Remaining × (its weight ÷ sum of all weights)   ← weights auto-normalize
ETA (debt/buy)   = (target − saved) ÷ monthly amount
```
"Typical spending" is the average of your completed months (or a projection of the current month
if that's all you have), with each category counted as fixed or flexible by its **kind**. Set your
paycheck, savings, and emergency-fund months in **Settings → Income & savings** first.

## Where to deploy it (works on any device — phone, tablet, computer)
This is a static site (just files), so any static host works and gives you **one HTTPS link you
open on every device**. All of these have a free tier; pick one:

| Host | How | Best for |
|------|-----|----------|
| **Netlify Drop** | Go to https://app.netlify.com/drop and **drag this folder** onto the page | Easiest, no account needed to start |
| **Cloudflare Pages** | Create a project, upload the folder (or connect a GitHub repo) | Fast, generous free tier |
| **Vercel** | `npx vercel` in this folder, or drag-drop in the dashboard | Simple if you use GitHub |
| **GitHub Pages** | Push the folder to a repo → Settings → Pages → deploy from branch | Free with a GitHub account |

After deploying you get a URL like `https://yourname.netlify.app`. Open it:
- **On your computer:** just use it in the browser (full sidebar layout), or install via the browser's "Install app" icon in the address bar.
- **Android (Chrome):** menu ⋮ → *Add to Home screen* / *Install app*.
- **iPhone (Safari):** Share → *Add to Home Screen*.

It then runs full-screen and works offline on each device.

> ⚠️ The same URL gives every device the **app**, but each device keeps its **own data** until you
> turn on **Cloud sync** below and sign in with the same account everywhere.

> Local testing only (no install): run `python -m http.server 8000` in this folder and open
> `http://<your-PC-IP>:8000`. Fine for trying it; offline-install needs HTTPS hosting.

## Cloud sync setup (optional — sync phone ↔ computer)
Sync uses **Supabase** (free tier). One-time setup, ~5 minutes:

1. Create a free account at **https://supabase.com** and make a **New project** (pick any name/password; wait ~1 min for it to start).
2. Open **Settings → API** for the **Project URL**, and **Settings → API Keys** for the
   **Publishable key**. (Supabase renamed the old "anon" key to **"Publishable key"** — it's the
   public, browser-safe one. Do **not** use the "Secret key", the "Direct connection string", or CLI setup.)
3. Paste both into **`config.js`** in this folder:
   ```js
   window.MT_SUPABASE_URL = 'https://YOURPROJECT.supabase.co';
   window.MT_SUPABASE_ANON_KEY = 'eyJhbGci...your-anon-key...';
   ```
   (The anon key is safe to ship publicly — Row Level Security below locks each user to their own data.)
4. In Supabase, open **SQL Editor → New query**, paste this, and **Run**:
   ```sql
   create table app_state (
     user_id uuid primary key references auth.users on delete cascade,
     data jsonb not null,
     updated_at timestamptz not null default now()
   );
   alter table app_state enable row level security;
   create policy "own row" on app_state for all
     using (auth.uid() = user_id) with check (auth.uid() = user_id);
   ```
5. (Optional, for the smoothest sign-up) In **Authentication → Providers → Email**, turn **off**
   "Confirm email" so you can sign in immediately without a confirmation link.
6. **Re-deploy / re-upload** the folder, then on each device open **Settings → Account & Sync** and
   **Create account** (once) / **Sign in** with the same email + password.

That's it — changes then sync automatically across devices. If two devices are edited while offline,
the **most recent change wins** (whole-data, last-write-wins). Your JSON/CSV exports still work as
extra backups.

## Where your data is stored (and how safe it is)
By default everything is saved in the browser's **localStorage** — on the device itself.

- **Permanent and free.** No subscription, no trial. Nothing expires and nothing is ever deleted
  "after free." The app requests **persistent storage** so the browser won't auto-evict it;
  *Settings → Data* shows the status and how much you've stored.
- **Private.** Without sign-in it never leaves the device. (Exceptions, only when you choose them:
  the optional OpenAI tips, and Cloud sync once you sign in — which stores your data in *your own*
  private Supabase account, readable only by you via Row Level Security.)
- **Local copy is gone if** you tap *Erase all data*, clear the browser's site data, uninstall and
  clear its data, or reset/lose the device — but if Cloud sync is on, your data is safe in the cloud
  and re-appears when you sign in again.

Because it's on the device, **the backup is your safety net**: use *Settings → Export JSON*
regularly (and **Import** to restore on a new phone). For spending records specifically, the
**Report → Export CSV** gives you a spreadsheet copy for any date range.

## Files
| File | Purpose |
|------|---------|
| `index.html` | The entire app (UI + logic) |
| `config.js` | Your Supabase URL + key for Cloud sync (blank = sync off) |
| `chart.umd.min.js` | Charts library, bundled for offline use |
| `manifest.json` | App name, icons, install config |
| `sw.js` | Service worker — caches everything for offline |
| `icon-*.png`, `apple-touch-icon.png` | Home-screen icons |

> If you change any cached file, bump `CACHE` in `sw.js` (currently `moneytrack-v4`) so phones pick up the update.
