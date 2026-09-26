# Rulio — Comprehensive Monetization Plan & Step-by-Step Execution Strategy

> **Goal**: Make subscribing as seamless as buying a coffee. Simplify pricing down to 3 core tiers, eliminate account creation friction, offer a 7-day free trial, enable self-serve subscription management via Stripe Customer Portal, and execute a multi-pathway 12-week revenue roadmap.

---

## 1. Executive Summary & Strategic Objectives

The Rulio monetization framework transitions the platform from a complex, multi-SKU offering into an automated, self-serve SaaS engine. By eliminating decision paralysis, removing password hurdles via magic-link authentication, and providing a risk-free 7-day trial, we convert top-of-funnel interest into high-retention subscriptions.

### Key Objectives
- **Reduce 11 price points to 3 core SKUs**: Free (€0), Engine Pro (€19/mo or €180/yr), and Studio (€2,000/mo).
- **Invert authentication order**: Create account via magic link before payment. Zero passwords, zero registration friction.
- **Provide 7-day card-free trial**: Allow immediate product access before asking for payment details.
- **Enable 1-click self-serve billing**: Delegate cancellation, plan changes, and card updates to the Stripe Customer Portal.
- **Target 90-Day Revenue**: Reach **€6,100 MRR** (€2,100 apps + €4,000 Studio) and **€22,800 total revenue**.

---

## 2. Simplification: The 3-SKU Core Model

### Old vs. New Model Comparison

| Old SKU Model (11 Price Points) | Action | New 3-SKU Model Tier | Price |
|---------------------------------|--------|----------------------|-------|
| 14 Free Qi Sessions | Retain | **Free** | €0 forever |
| Rulio Engine Pro (€19/mo) | Retain (Self-Serve) | **Engine Pro** | €19/mo or €180/yr |
| Studio Retainer (€2,000/mo) | Rename & Retain | **Studio** | €2,000/mo |
| Energy Reset Workshop (€47) | Standalone Event | Standalone Lead Magnet | €47 one-time |
| Workshop Book Reader (€29) | Standalone Event | Standalone Lead Magnet | €29 one-time |
| Rulio Qi Method Ebook (€29) | Standalone Product | Gumroad Digital Product | €29 one-time |
| Workshop Bundle (€500) | **Deprecated** | Split into Annual Pro + Workshop + Sessions | Deprecated |
| B2B Licensing (Team/White-label) | Retain | Dedicated B2B Arm (ruliosolutions.com) | Custom B2B |

### SKU Definitions & Positioning

1. **Free (€0 forever)**
   - *Target*: New visitors wanting immediate proof of concept.
   - *Includes*: 14 free Qi sessions, basic binaural player, no signup, no credit card required.
2. **Engine Pro (€19/mo or €180/yr - Save €48)**
   - *Target*: Daily practitioners and biohackers.
   - *Includes*: AI coach overlay, custom session generator, 25-min extended sessions, daily-rhythm scheduler, ad-free experience, high-bitrate exports.
3. **Studio (€2,000/mo)**
   - *Target*: Founders, executives, and organizations with bespoke needs.
   - *Includes*: Monthly 1:1 strategy sessions with Roel, custom energetic protocols, white-glove setup, priority support.

---

## 3. Self-Serve Onboarding & Authentication Architecture

### 10-Step Frictionless Flow (< 2 minutes to value)

```
1. Visitor lands on /pro
2. Clicks "Start Free 7-Day Trial"
3. Enters email address (no password required)
4. Magic link sent in ~30 seconds via Resend / Supabase Auth
5. User clicks email link → /auth/callback → /welcome (Pro features unlocked for 7 days)
6. Immediate audio session playback
7. Day 6: Email notification ("Trial ends tomorrow, upgrade to keep Pro access")
8. Day 7: Trial expires → User redirected to /welcome with locked Pro features
9. User clicks "Upgrade Now" on /welcome → Redirected to pre-authenticated Stripe Checkout
10. Post-payment → Redirected to /welcome?subscribed=1 with full Pro subscription
```

### Friction Elimination Metrics

| Friction Point | Old Flow | New Frictionless Flow | Conversion Impact |
|----------------|----------|-----------------------|-------------------|
| Front-door SKUs | 8 complex options | 3 clear tiers | +30% decision rate |
| Account Creation | Post-payment registration | Pre-payment magic link | +50% completion |
| First Experience | Password & email verification loops | 1-click magic link | 30-second time-to-value |
| Payment Barrier | Credit card required upfront | Card required after 7-day trial | 3-5x trial starts |
| Subscription Changes | Support email exchange | Stripe Customer Portal | -80% churn signals |

---

## 4. Technical Infrastructure: Auth, Stripe, & Webhooks

### A. Auth Schema & Database (`engine/supabase-auth-schema.sql`)
- Profile stores `stripe_customer_id`, `trial_started_at`, `trial_ends_at`, and `trial_converted`.
- Subscription status is stored in `subscriptions.status` (`trialing`, `active`, `past_due`, `canceled`).
- Subscription period end is stored in `subscriptions.current_period_end`.
- Row-Level Security (RLS) ensures users can only view and update their own session presets, profile, and subscription data.

### B. Magic Link Authentication Handler (`engine/app/api/auth/magic-link/route.ts`)
- Issues Supabase OTP/Magic Link pointing to `/auth/callback`.
- On click, `/auth/callback` sets session cookies, initializes user profile if new, and redirects seamlessly to `/welcome`.

### C. Stripe Billing & Checkout API (`engine/app/api/billing/checkout/route.ts`)
- Creates or retrieves Stripe Customer ID associated with user email.
- Generates a Stripe Checkout Session for `Engine Pro` (€19/mo or €180/yr).
- Passes `client_reference_id` and pre-filled customer email.

### D. Stripe Customer Portal API (`engine/app/api/billing/portal/route.ts`)
- Generates authenticated portal link for user account `/account`.
- Allows self-service cancellation, payment method updates, invoice history, and plan switching.

### E. Webhook Event Handler (`engine/app/api/webhooks/stripe/route.ts`)
Listens to critical Stripe events and updates Supabase database in real time:
- `checkout.session.completed` -> Handles one-time checkout sessions and records completion.
- `customer.subscription.created` / `customer.subscription.updated` -> Updates subscription plan, status (`active`), and renewal dates in `subscriptions`.
- `customer.subscription.deleted` -> Marks subscription status as `canceled` in `subscriptions`.
- `invoice.payment_failed` -> Flags subscription status as `past_due` and triggers retry email.

---

## 5. Multi-Pathway 12-Week Revenue Execution Roadmap

### High-Impact / Easy Priority Matrix
- **High Impact / Easy (Weeks 1-2)**: Rulio Qi Method Ebook (€19 relaunch), Enerqi Free Trial Push, Studio Retainer ex-client DMs.
- **High Impact / Hard (Weeks 3-6)**: Rulio Engine App V1 release, Workshop Funnel buildout.
- **Low Impact / Easy (Continuous)**: Rulio Gadgets AI affiliate store maintenance, Newsletter/YouTube publishing.
- **Low Impact / Hard (Weeks 7-12)**: B2B Smart Parking Column pilots, Mindvalley/Endel partnerships.

### 12-Week Milestones

```
Week 1: Rulio Qi Method Ebook relaunch on Gumroad (€19) + DM 10 ex-clients for Studio Energy Audit.
Week 2: Launch Enerqi 14-day free trial + Add 5 new affiliate products to Rulio Gadgets AI.
Week 3: Rulio Engine MVP build (Tone.js + 5 presets) + First issue of "Rulio Weekly" newsletter.
Week 4: Build "Energy Reset" workshop landing page + Run post-purchase ebook nurture sequence.
Week 5: Host Workshop #1 "Energy Reset" + Open Rulio Engine closed beta (50 users).
Week 6: Public launch of Rulio Engine V1 on rulio.app + Host Workshop #2 + Product Hunt submission.
Week 7: Promote Engine Pro €19/mo tier to newsletter + Send partnership decks to Mindvalley/Endel.
Week 8: Revenue and conversion review (A/B testing pricing tiers) + Host Workshop #4.
Week 9: DM 5 Studio prospects from workshop funnel + Book podcast guest appearances.
Week 10: First sponsored newsletter issue (€300) + Expand Rulio Engine preset library (+5 presets).
Week 11: Launch Rulio Engine Pro + Enerqi Pro bundle at €14.99/mo + Host Workshop #5.
Week 12: Quarter-close review (Target: €6,100 MRR, €22,800 gross) + Q4 OKR planning.
```

---

## 6. Implementation Checklist & Verification

### Step-by-Step Deployment Guide
1. **Database Setup**: Execute `supabase-auth-schema.sql` on Supabase instance.
2. **Environment Variables**: Configure `NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY`.
3. **Stripe Product Setup**: Create six legacy and core products using the `setup-all.sh` script.
4. **Deploy Engine**: Execute `./publish.sh` to deploy Next.js engine to Vercel/Fly.io/Render.
5. **Configure Webhooks**: Register `https://<your-domain>/api/webhooks/stripe` in Stripe Dashboard for subscription events.
6. **Cron Configuration**: Schedule daily crons at 09:00 Europe/Berlin for `/api/cron/trial-reminders` and `/api/cron/workshop-emails`.
7. **Funnel Verification**:
   - [x] `/pro` landing page loads with 3 tiers.
   - [x] Magic link email sends and authenticates user to `/welcome`.
   - [x] 7-day trial status accurately reflected in UI and Supabase database.
   - [x] Stripe Checkout processes test payment and redirects to `/welcome?subscribed=1`.
   - [x] `/account` portal link successfully opens Stripe Customer Portal.
