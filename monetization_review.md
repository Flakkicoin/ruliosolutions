# Rulio — monetization review & simplification

> **Goal**: make subscribing as easy as buying a coffee. Simplify the self-serve front door to three clear tiers, remove account-creation friction, offer a 7-day free trial, and let customers manage billing through Stripe Customer Portal.

## What we had (the old funnel)

11 SKUs across the customer journey:

| # | SKU | Price | Where it lives |
|---|-----|-------|----------------|
| 1 | Free Qi sessions (14) | €0 | rulio.app/qi |
| 2 | Rulio Engine Pro | €19/mo | Mentioned in bundles |
| 3 | The Rulio Qi Method book | €29 | rulio.io |
| 4 | Energy Audit | Free | rulio.io/audit |
| 5 | Energy Reset Workshop — Standard | €47 | rulio.io/workshop |
| 6 | Energy Reset Workshop — Book Reader | €29 | rulio.io/workshop |
| 7 | Energy Reset Bundle | €500 | rulio.io/workshop |
| 8 | Studio Retainer | €2,000/mo | Sales only |
| 9 | Team licence (B2B) | €19/seat/mo | ruliosolutions.com |
| 10 | White-label (B2B) | from €4,500/mo | ruliosolutions.com |
| 11 | Platform integration (B2B) | from €12,000/mo | ruliosolutions.com |

The self-serve front door is reduced to three tiers: **Free, Engine Pro, and Studio**. The book, workshop, audit, and B2B offerings remain separate supporting or sales-led products.

## What's broken

1. **No self-serve Engine Pro page.** The €19/mo product exists in Stripe, but there is no public page that subscribes users to it.
2. **Account creation happens after payment.** The old flow asks customers to pay, then create and verify an account. The new flow authenticates first with a passwordless magic link.
3. **No free trial.** A card-free seven-day trial lets customers experience Pro before payment.
4. **No subscription management.** Stripe Customer Portal removes the need to email support to cancel, update a card, or change plans.
5. **Pricing is confusing.** The self-serve page should make the intended choice obvious without hiding the existing workshop, book, Studio, or B2B paths.

## The new model (3 self-serve tiers)

| Tier | Price | What you get |
|------|-------|--------------|
| **Free** | €0 forever | 14 Qi sessions, no signup, no card |
| **Engine Pro** | **€19/mo** or **€180/yr** (save €48) | AI coach, custom session generator, 25-minute extended sessions, daily-rhythm scheduler, no ads |
| **Studio** | **€2,000/mo** | Monthly 1:1 with Roel, custom protocols, white-glove setup, priority support |

- **Free** → “I want to try it.”
- **Engine Pro** → “I use it every day and want the advanced features.”
- **Studio** → “I have a difficult problem and want human support.”

## What happens to the other offerings?

- **The Rulio Qi Method book** stays as a €29 one-time product on Gumroad.
- **Energy Reset Workshop (€47)** stays as a one-time event and feeds Studio and Engine Pro.
- **Bundle (€500)** is deprecated. The proposed split offering—annual Pro (€180) + workshop (€47) + two private sessions (€400)—costs **€627**, which is €127 more than the old bundle; it should not be marketed as a half-price replacement.
- **Book Reader (€29)** stays as a workshop-specific option.
- **Studio Retainer (€2,000/mo)** becomes the Studio tier name.
- **Team, White-label, and Platform B2B offerings** remain on ruliosolutions.com.

## The new self-serve flow

The flow has 10 system and user steps, but only four primary user actions: **visit, submit email, click the magic link, and pay**.

```text
1. User visits /pro.
2. Clicks “Start free 7-day trial”.
3. Enters an email address.
4. Supabase sends a magic link.
5. User clicks the link → /auth/callback → /welcome.
6. Pro access is unlocked for the seven-day trial.
7. Day 6: send “trial ends tomorrow” email.
8. Day 7: lock Pro features while leaving the account and free sessions available.
9. User clicks Upgrade → authenticated Stripe Checkout.
10. Payment completes → /welcome with full Pro access.
```

There is no password, post-payment account-creation page, or separate email-verification loop.

## Friction removed

| Old | New |
|-----|-----|
| 11 SKUs across the journey; 8 front-door choices | 3 self-serve tiers |
| Create account after payment | Authenticate before payment with a magic link |
| Card required upfront | Card required only after the seven-day trial |
| Email support to cancel or update billing | Stripe Customer Portal |
| Multiple unclear conversion paths | `/pro` → trial → paid Engine Pro |

## Conversion math

These are planning assumptions, not guarantees:

- Visitor → trial start: 5–10%
- Trial start → trial completion: 40–60%
- Trial completion → paid: 15–25%
- End-to-end visitor → paid: approximately 0.3–1.5%

For 1,000 `/pro` visitors per month, that implies 50–100 trial starts and approximately 3–15 paid Engine Pro subscribers, or €57–€285 MRR before churn.

## The pricing page

The page lives at `/pro` on the engine: single-column on mobile and three tiers side-by-side on desktop. A **single shared trial form** serves the Engine Pro tier rather than requiring a separate card-level CTA. The Engine Pro tier is marked “Most popular.” Free and Studio retain their existing informational or sales paths.

## Implementation order

Run the database schemas in dependency order before configuring the application:

| Step | Time | Status |
|------|------|--------|
| Run `engine/supabase-schema.sql` | 2 min | Code ready |
| Run `engine/supabase-auth-schema.sql` | 2 min | Code ready; depends on the funnel schema |
| Configure Supabase Auth (magic link and redirect URLs) | 3 min | Guide ready |
| Create Stripe products and prices for the three tiers | 5 min | Guide ready |
| Deploy `/pro` self-serve page | 2 min | Code ready |
| Deploy `/welcome` post-magic-link page | 1 min | Code ready |
| Deploy `/account` subscription-management page | 2 min | Code ready |
| Update Stripe webhook for subscriptions | 5 min | Code ready |
| Update PostHog events for the new flow | 1 min | Code ready |
| **Total** | **~25 min** | **All code in the repo** |

## The front door

```text
Free (rulio.app/qi)              ← 14 free sessions, no signup
  ↓
/pro (rulio.app/pro)              ← 7-day free trial, no card
  ↓
Engine Pro (€19/mo)               ← self-serve, Stripe Customer Portal
  ↓
Studio (€2,000/mo)                ← sales-led, custom protocol

The book (rulio.io)               ← €29 one-time
  ↓
The workshop (rulio.io/workshop)  ← €47 one-time
  ↓
The audit (rulio.io/audit)        ← free 30-minute call
```

## The B2B arm

Team, White-label, and Platform offerings remain on ruliosolutions.com. Engine Pro is the self-serve starting point for individuals and small teams; larger teams should be routed to the B2B Team licence through the sales path.

## Recap

| | Old | New |
|---|---|---|
| Self-serve tiers | 8 front-door choices | 3 |
| Subscription journey | 11 legacy steps and support handoffs | 4 primary user actions |
| Free trial | No | 7 days, no card |
| Subscription management | Support email | Stripe Customer Portal |
| Account creation | After payment | Before payment via magic link |

## Source of truth

- Setup guide: `supabase_setup.md`
- Funnel schema: `engine/supabase-schema.sql`
- Auth schema: `engine/supabase-auth-schema.sql`
- Magic-link send: `engine/app/api/auth/magic-link/route.ts`
- Subscription checkout: `engine/app/api/billing/checkout/route.ts`
- Stripe portal: `engine/app/api/billing/portal/route.ts`
- Stripe webhook: `engine/app/api/webhooks/stripe/route.ts`
- Self-serve page: `engine/app/pro/page.tsx`
- Account page: `engine/app/account/page.tsx`
- Auth callback: `engine/app/auth/callback/route.ts`
