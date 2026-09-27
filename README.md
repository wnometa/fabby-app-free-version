# FABBY

**AI-assisted bookkeeping for small businesses.** FABBY helps you record income and expenses, keep business and personal money separate, and understand where your business stands — in plain language.

Built by **Wiltecon Technologies**. Lead developer: **Wilson Nometa**.

© Wiltecon Technologies. All rights reserved. This software and its source code are the property of Wiltecon Technologies and may not be copied, distributed, or used without permission.

---

## Table of contents

1. [What's in this app](#whats-in-this-app)
2. [Tech stack](#tech-stack)
3. [Prerequisites](#prerequisites)
4. [Local setup](#local-setup)
5. [Setting up Supabase](#setting-up-supabase)
6. [Environment variables](#environment-variables)
7. [Running locally](#running-locally)
8. [Building for production](#building-for-production)
9. [Deploying](#deploying)
10. [Post-deploy checklist](#post-deploy-checklist)
11. [Project structure](#project-structure)
12. [Known limitations (v1)](#known-limitations-v1)
13. [Support](#support)

---

## What's in this app

FABBY (v1) covers:

- **Email/password authentication** — sign up, log in, email verification, password reset (via Supabase Auth)
- **Income** — record and manage income transactions
- **Expenses** — record and manage business/personal expenses
- **Invoices** — track customer invoices and payment status
- **Business vs Personal** — separate business and personal money, with mixing warnings
- **Insights** — a summarized view of financial activity
- **Reports** — profit & loss, income, expense, and other report views over custom date ranges
- **Settings** — user preferences
- **Ask FABBY** — a natural-language assistant that turns a sentence like *"Spent ₦12k on delivery"* into a categorized transaction

All financial data (income, expenses, invoices) is stored in **Supabase Postgres**, scoped per user with Row Level Security — not in browser storage — so it's available from any device you log in from.

## Tech stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router, Turbopack) |
| UI | React 19, Tailwind CSS 4 |
| Backend / DB | Supabase (Postgres, Auth, Row Level Security) |
| Language | TypeScript |
| Hosting (recommended) | Vercel |

## Prerequisites

- **Node.js 20+** and npm
- A **Supabase** account and project ([supabase.com](https://supabase.com))
- A **Vercel** account (or any Node-compatible host) for deployment
- The **Supabase CLI** (optional, but recommended for running migrations) — [install instructions](https://supabase.com/docs/guides/cli/getting-started)

## Local setup

```bash
# 1. Install dependencies
npm install

# 2. Create your local env file (see "Environment variables" below for the values)
touch .env.local

# 3. Run the database migrations against your Supabase project (see next section)

# 4. Start the dev server
npm run dev
```

The app will be available at [http://localhost:3000](http://localhost:3000).

## Setting up Supabase

1. **Create a project** at [supabase.com/dashboard](https://supabase.com/dashboard).
2. **Get your API credentials**: Project Settings → API → copy the `Project URL` and the `anon public` key. You'll need these for `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
3. **Run the migrations** in `supabase/migrations/` against your project, in order. This creates the `profiles`, `transactions`, `invoices`, `goals`, `goal_contributions`, and `user_settings` tables with Row Level Security enabled, so each user can only see their own data.

   Using the Supabase CLI:
   ```bash
   supabase link --project-ref <your-project-ref>
   supabase db push
   ```
   Or, run each `.sql` file in `supabase/migrations/` manually via the Supabase Dashboard → SQL Editor, in filename (chronological) order.

4. **Configure Auth email settings** — Authentication → URL Configuration in the Supabase Dashboard:
   - **Site URL**: your production URL (e.g. `https://your-domain.com`) — use `http://localhost:3000` while developing locally.
   - **Redirect URLs**: add both `http://localhost:3000/auth/callback` (local) and `https://your-domain.com/auth/callback` (production). The app uses `/auth/callback` to complete email verification and password-reset links.
   - Decide whether **"Confirm email"** is required before login (Authentication → Providers → Email). If it's on, users must click the link in their confirmation email before they can log in — the app already handles this flow via `/verify-email`.

## Environment variables

Create a `.env.local` file (for local development) with:

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Yes | Your Supabase project URL, e.g. `https://xxxxx.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Yes | Your Supabase project's public `anon` key |
| `NEXT_PUBLIC_APP_URL` | Yes | The full base URL of the deployed app, e.g. `https://fabby-app.com`. Used to build email verification / password-reset redirect links. Falls back to `http://localhost:3000` if unset. |

None of these are secret enough to require server-only handling (the `anon` key is safe to expose to the browser — access is enforced by Row Level Security in Postgres, not by hiding this key), but they must still be set correctly per environment.

## Running locally

```bash
npm run dev       # start the dev server (Turbopack)
npm run lint       # run ESLint
```

## Building for production

```bash
npm run build      # production build
npm run start       # serve the production build locally, for a final check before deploying
```

Confirm this completes with no errors before deploying.

## Deploying

**Recommended: Vercel** (this is a standard Next.js App Router project, no special configuration needed).

1. Push this repository to GitHub.
2. In Vercel, "Add New Project" → import the repository.
3. Add the three environment variables from the table above under Project Settings → Environment Variables (set them for both **Production** and **Preview** environments — use your production Supabase project for Production and, ideally, a separate Supabase project for Preview/staging).
4. Deploy.
5. Once you have your live Vercel URL (or custom domain), go back into Supabase → Authentication → URL Configuration and update the **Site URL** and **Redirect URLs** to match it, and update `NEXT_PUBLIC_APP_URL` in Vercel to match as well.

If deploying elsewhere (Railway, Render, a VPS, etc.), the requirements are the same: Node.js 20+, the three env vars set, and `npm run build && npm run start`.

## Post-deploy checklist

Run through this after every deploy, and definitely before sharing the app with real users:

- [ ] `supabase/.temp/` is **not** committed to the repository — it contains local Supabase CLI session data (project ref, pooler URL) that shouldn't be in version control. Add `supabase/.temp/` to `.gitignore` if it isn't already, and remove it from git history if it was previously committed.
- [ ] Supabase **Site URL** and **Redirect URLs** point to your real production domain, not `localhost`.
- [ ] `NEXT_PUBLIC_APP_URL` in your hosting provider matches your real production domain.
- [ ] Sign up a fresh test account on the live URL and confirm: the verification email arrives, the link works, and you can log in afterward.
- [ ] Add one income entry, one expense, one invoice, and confirm they persist after a full page reload and after logging out and back in.
- [ ] Log in with a second test account and confirm it does not see the first account's data (Row Level Security check).
- [ ] Try "Ask FABBY" and confirm a described transaction actually gets saved and shows up in Income/Expenses.

## Project structure

```
src/
  app/
    dashboard/        Main authenticated app shell (Income, Expenses, Invoices, Reports, Insights, Business vs Personal, Settings, Ask FABBY)
    login/, signup/, forgot-password/, reset-password/, verify-email/, auth/callback/   Auth pages
    layout.tsx         Root layout, fonts, metadata
  components/          Feature modules (income, expenses, insights, reports, settings, money-separation, etc.)
  contexts/            React context providers (auth-context.tsx)
  lib/
    auth.ts             Supabase Auth helper functions
    supabase/           Browser and server Supabase client factories + generated DB types
    goals.ts, income.ts, financial-ledger.ts, settings.ts, insights-data.ts   Domain logic
  middleware.ts         Route protection (redirects unauthenticated users away from the app, and authenticated users away from auth pages)
supabase/
  migrations/           SQL migrations — run these against your Supabase project before first use
```

## Known limitations (v1)

Being upfront about what's still rough in this first version:

- **Settings and Goals** currently persist to browser `localStorage` rather than Supabase, so they won't sync across devices or survive a cleared browser yet. Everything financial (income, expenses, invoices) is fully database-backed.
- There's a leftover `reports-module-old.tsx` file in `src/components/` that isn't used by the app — safe to delete, kept for now in case anything needs to be cross-referenced.
- "Ask FABBY" uses lightweight keyword/regex parsing to interpret what you type, not a full language model — it works well for direct statements ("Sold 3 shirts for ₦45k") but can misread ambiguous phrasing.
- The `middleware.ts` file uses Next.js's `middleware` convention, which Next.js has marked deprecated in favor of `proxy.ts`. It still works correctly today; migrating is a good idea before Next.js removes support entirely.

None of these block using the app for real bookkeeping — they're flagged here so they're a deliberate, known backlog rather than a surprise.

## Support

For issues, questions, or feature requests, contact **Wiltecon Technologies**.

---

*FABBY is a product of Wiltecon Technologies. All rights, title, and interest in and to this software, including all intellectual property rights, are owned exclusively by Wiltecon Technologies.*
