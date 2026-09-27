# Rulio Engine — Deployment Guide

> **Goal**: Deploy the self-serve Engine Pro funnel to production in ~45 minutes.
> This guide assumes you have already reviewed `monetization_review.md` and `supabase_setup.md`.

## Prerequisites

- **Supabase account**: https://app.supabase.com
- **Stripe account** (test mode is fine to start): https://dashboard.stripe.com
- **Resend account** (for transactional email): https://resend.com
- **Node.js 18+** and npm
- **A domain** or deployed engine URL (e.g., `https://rulio.app`)

## Phase 1: Database Setup (5 minutes)

### 1.1 Create a Supabase project

1. Go to https://app.supabase.com → **New project**
2. Name it `rulio-production` (or `rulio-staging` for development)
3. Set a strong database password (save it to your password manager)
4. Region: **West EU (Ireland)** or **Frankfurt** (geographically closest to Brussels)
5. Plan: Free tier works for development; upgrade to Pro when you hit 50k MAU
6. Click **Create new project** — wait ~2 minutes for provisioning

### 1.2 Run the funnel schema

1. In Supabase, go to **SQL Editor** (left sidebar)
2. Click **New query**
3. Copy the entire contents of `engine/supabase-schema.sql` from the repo
4. Paste into the editor and click **Run**
5. Expected: "Success. No rows returned" ✓

### 1.3 Run the auth schema

1. Click **New query** again
2. Copy the entire contents of `engine/supabase-auth-schema.sql`
3. Paste and click **Run**
4. Expected: "Success. No rows returned" ✓

### 1.4 Verify the tables

In Supabase, go to **Table Editor** (left sidebar). You should see **7 tables**:

| Table | Purpose |
|-------|---------|
| `workshop_attendees` | Workshop ticket purchases |
| `audit_leads` | Free audit registrations |
| `email_events` | Email delivery logs |
| `profiles` | User accounts (mirrors auth.users) |
| `subscriptions` | Engine Pro, Studio, workshop subscriptions |
| `sessions_log` | Qi session playback history |
| `magic_link_events` | Magic-link auth tracking |

If all 7 appear, **proceed to Phase 2**.

## Phase 2: Authentication Setup (3 minutes)

### 2.1 Configure magic-link email

1. Go to **Authentication → Providers** (left sidebar)
2. Click **Email** (enabled by default)
3. **Confirm email** → turn **OFF** (we use the magic link as verification)
4. **Secure email change** → turn **ON** (industry standard)
5. Save

### 2.2 Set auth redirect URLs

1. Go to **Authentication → URL Configuration**
2. **Site URL**:
   - Local dev: `http://localhost:3000`
   - Staging: `https://rulio-engine.fly.dev` (or your staging domain)
   - Production: `https://rulio.app`
3. **Redirect URLs** (one per line):
   ```
   http://localhost:3000/auth/callback
   https://rulio-engine.fly.dev/auth/callback
   https://rulio.app/auth/callback
   ```
4. Save

### 2.3 Get your API keys

1. Go to **Settings → API** (left sidebar)
2. Copy these three values:

| Value | Environment variable |
|-------|----------------------|
| **Project URL** (e.g., `https://abcxyz.supabase.co`) | `NEXT_PUBLIC_SUPABASE_URL` |
| **anon public key** (long `eyJ...` string) | `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| **service_role key** (click Reveal) | `SUPABASE_SERVICE_ROLE_KEY` |

## Phase 3: Stripe Setup (5 minutes)

### 3.1 Create Engine Pro products

1. Go to https://dashboard.stripe.com/products
2. Click **Add product**
3. Fill in each product:

#### Product 1: Rulio Engine Pro — Monthly
- **Name**: `Rulio Engine Pro ��� Monthly`
- **Type**: Recurring
- **Price**: €19.00 EUR, monthly
- Copy the **Price ID** (looks like `price_xyz123`)
- Set environment variable: `STRIPE_PRICE_ENGINE_PRO_MONTHLY=price_xyz123`

#### Product 2: Rulio Engine Pro — Annual
- **Name**: `Rulio Engine Pro — Annual`
- **Type**: Recurring
- **Price**: €180.00 EUR, yearly
- Copy the **Price ID**
- Set: `STRIPE_PRICE_ENGINE_PRO_ANNUAL=price_xyz456`

#### Product 3: Rulio Studio — Monthly
- **Name**: `Rulio Studio — Monthly`
- **Type**: Recurring
- **Price**: €2,000.00 EUR, monthly
- Copy the **Price ID**
- Set: `STRIPE_PRICE_STUDIO_MONTHLY=price_xyz789`

### 3.2 Get Stripe API keys

1. Go to https://dashboard.stripe.com/apikeys
2. Copy:
   - **Publishable key** → `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`
   - **Secret key** → `STRIPE_SECRET_KEY`

### 3.3 Create a webhook endpoint (for later)

1. Go to https://dashboard.stripe.com/webhooks
2. Click **Add endpoint**
3. **Endpoint URL**: `https://rulio.app/api/webhooks/stripe` (your production domain)
4. **Events to listen**: select all of these:
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
   - `checkout.session.completed`
   - `charge.refunded`
5. Click **Add endpoint**
6. Copy the **Signing secret** (looks like `whsec_xyz...`)
7. Set: `STRIPE_WEBHOOK_SECRET=whsec_xyz...`

## Phase 4: Email Setup (2 minutes)

### 4.1 Configure Resend

1. Go to https://resend.com
2. Sign up or log in
3. Go to **API Keys** and create a new key
4. Copy the key → `RESEND_API_KEY`
5. Verify your domain (or use Resend's default domain for testing)

## Phase 5: Application Environment (2 minutes)

### 5.1 Create `.env.local` in `engine/`

Create a file `engine/.env.local` with all the keys from Phases 2–4:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://abcxyz.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ENGINE_PRO_MONTHLY=price_xyz123
STRIPE_PRICE_ENGINE_PRO_ANNUAL=price_xyz456
STRIPE_PRICE_STUDIO_MONTHLY=price_xyz789

# Resend
RESEND_API_KEY=re_...

# Application
NEXT_PUBLIC_URL=https://rulio.app
NODE_ENV=production
```

## Phase 6: Local Testing (5 minutes)

### 6.1 Install and run

```bash
cd engine
npm install
npm run dev
```

The engine runs on `http://localhost:3000`.

### 6.2 Test the magic link flow

**Test 1: Send a magic link**

```bash
curl -X POST http://localhost:3000/api/auth/magic-link \
  -H "content-type: application/json" \
  -d '{"email":"you@example.com","firstName":"You"}'
```

Expected: `{ "ok": true }` (don't reveal success/failure for security)

**Test 2: Check your email**

You should receive an email from Resend within 30 seconds with a "Sign in to Rulio" button.

**Test 3: Click the link**

The link points to `http://localhost:3000/auth/callback?code=...`. You should land on `/welcome`.

**Test 4: Check the database**

In Supabase → **Table Editor** → **profiles**, you should see a new row with your email and `trial_ends_at` set to 7 days from now.

### 6.3 Test the checkout flow

**Test 5: Attempt to create a checkout session**

```bash
# First, get an auth cookie by clicking the magic link
# Then use the authenticated session to hit checkout:

curl -X POST http://localhost:3000/api/billing/checkout \
  -H "content-type: application/json" \
  -d '{"plan":"engine_pro_monthly"}' \
  -H "cookie: <your session cookie>"
```

Expected: `{ "url": "https://checkout.stripe.com/pay/..." }`

**Test 6: Go through Stripe Checkout**

Use Stripe's test card: `4242 4242 4242 4242`, any future expiry, any CVC.

Expected after payment:
- Redirect to `http://localhost:3000/welcome?subscribed=1`
- A new row in `subscriptions` table with `status='active'`
- A PostHog event: `subscription_started`

## Phase 7: Deployment (20 minutes)

### 7.1 Deploy the engine

Choose one: **Vercel**, **Fly.io**, or **Render**.

#### Vercel (recommended for Next.js)

1. Push your code to GitHub
2. Go to https://vercel.com → **Import project**
3. Connect your repo
4. Set environment variables (paste from `.env.local`)
5. Deploy
6. Your app is live at `https://rulio.vercel.app` (or your custom domain)

#### Fly.io

1. Install `flyctl`: https://fly.io/docs/hands-on/install-flyctl/
2. In the `engine/` directory:
   ```bash
   flyctl launch
   ```
3. Set secrets from `.env.local`:
   ```bash
   flyctl secrets set SUPABASE_SERVICE_ROLE_KEY=... STRIPE_SECRET_KEY=... etc.
   ```
4. Deploy:
   ```bash
   flyctl deploy
   ```

### 7.2 Update Supabase redirect URLs

1. Go to Supabase → **Authentication → URL Configuration**
2. Add your production domain:
   ```
   https://rulio.app/auth/callback
   ```

### 7.3 Update Stripe webhook

1. Go to https://dashboard.stripe.com/webhooks
2. Edit the endpoint you created earlier
3. Change **Endpoint URL** to your production domain:
   ```
   https://rulio.app/api/webhooks/stripe
   ```

### 7.4 Test production

1. Visit https://rulio.app/pro
2. Enter an email and click **Start free trial**
3. Check your email for the magic link
4. Click it → should redirect to `/welcome`
5. Verify a trial profile exists in Supabase
6. Go to `/welcome`, click **Upgrade**
7. Go through Stripe Checkout with the test card
8. Verify the subscription appears in Supabase

### 7.5 Switch to live Stripe keys

When you're confident everything works:

1. Go to https://dashboard.stripe.com/apikeys
2. Toggle **Viewing test data** to OFF
3. Copy the live **Secret key** and **Publishable key**
4. Update your deployment's environment variables:
   - `STRIPE_SECRET_KEY` → live key
   - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` → live key
5. Re-deploy your application
6. Do **one test transaction** with a real €1 payment, then refund it
7. Verify the transaction and subscription appear in Stripe Dashboard

## Phase 8: Monitoring (ongoing)

### 8.1 Check logs

- **Vercel**: https://vercel.com → your project → **Deployments** → view logs
- **Fly.io**: `flyctl logs`
- **Application logs**: Supabase webhooks, Stripe events, Resend delivery

### 8.2 PostHog events

Set up a PostHog project to track user behavior:

- `magic_link_sent` — magic link was sent
- `subscription_checkout_started` — user started checkout
- `subscription_started` — payment succeeded
- `trial_ends_at` — profile `trial_ends_at` timestamp for trial duration analytics

### 8.3 Revenue tracking

Track in Stripe Dashboard:

- **MRR** (Monthly Recurring Revenue) = sum of active subscriptions / 12 or annual / 12
- **Trial-to-paid conversion** = subscriptions.status='active' / profiles with trial_ends_at < now() that exist in subscriptions
- **Churn rate** = canceled subscriptions / total subscriptions per month

## Troubleshooting

### "Invalid API key" when sending a magic link

→ Verify `SUPABASE_SERVICE_ROLE_KEY` (not anon) is set in `.env.local`.

### "User not found" after clicking the magic link

→ Verify Supabase **Authentication → URL Configuration** has the correct **Redirect URLs** for your domain.

### Stripe Checkout shows "Invalid session"

→ Verify `STRIPE_SECRET_KEY` and `STRIPE_PRICE_ENGINE_PRO_MONTHLY` are set and correct.

### Webhook events not showing up in Supabase

→ Check that `STRIPE_WEBHOOK_SECRET` is set. Verify the endpoint is registered in Stripe Dashboard and receiving events.

### Emails not sending

→ Check `RESEND_API_KEY`. If using a custom domain, verify it's verified in Resend. Check Resend Dashboard for delivery logs.

## Next Steps

1. **Monitor the trial-to-paid funnel** for the first week. Look for:
   - Trial starts per day
   - Trial completion rate (did they use the trial for 7 days?)
   - Trial-to-paid conversion rate

2. **Set up cron jobs** for recurring tasks:
   - `/api/cron/trial-reminders` — send "trial ends tomorrow" on day 6
   - `/api/cron/workshop-emails` — send workshop confirmation/reminder emails

3. **Gather feedback** from the first 10 paid subscribers on Slack or email.

4. **Plan expansion**:
   - Add more Qi sessions
   - Build the AI coach overlay
   - Launch Studio applications
   - Build the workshop funnel

---

**Source of truth for each component**:

- Funnel schema: `engine/supabase-schema.sql`
- Auth schema: `engine/supabase-auth-schema.sql`
- `/pro` page: `engine/app/pro/page.tsx`
- Magic link: `engine/app/api/auth/magic-link/route.ts`
- Checkout: `engine/app/api/billing/checkout/route.ts`
- Stripe webhook: `engine/app/api/webhooks/stripe/route.ts`
- Auth callback: `engine/app/auth/callback/route.ts`
- Monetization review: `monetization_review.md`
- Setup guide: `supabase_setup.md`
