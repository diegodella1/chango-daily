---
title: "Supabase + Stripe: simple, reliable customer syncing on your own terms"
date: 2026-03-15
tags: [supabase, stripe, automation]
---

If you run subscriptions on Stripe and accounts on Supabase, you eventually need a boring thing: keep customers in sync, deterministically. No flaky webhooks, no hidden state. Here’s a minimal pattern that works on a Raspberry Pi as well as in prod.

The goal
- Single source of truth: Stripe is the billing source; Supabase mirrors key fields.
- Deterministic reconciliation: a daily job fixes drift; webhooks provide immediacy.
- Idempotent writes: safe to retry everything.

Data shape (minimal)
- Supabase table: billing_customers
  - id UUID (PK, = auth.users.id)
  - stripe_customer_id text unique not null
  - email text not null
  - status text (active, past_due, cancelled)
  - updated_at timestamptz

Core flows
1) On signup: create a Stripe customer with metadata { supabase_user_id } and email. Store stripe_customer_id in billing_customers.
2) On subscription change: update status via webhook → tiny handler updates the row.
3) Nightly reconcile (cron): list Stripe customers updated in last 24h, upsert into Supabase. Cross-checks catch dropped webhooks or manual edits.

Why this survives reality
- Stripe is eventual but consistent; their API is reliable. Webhooks can drop; cron repair closes gaps.
- Idempotency keys on writes keep handlers safe.
- Email changes are tricky: use Stripe customer email as authority unless the user updates email in app → then patch Stripe in the same transaction.

Minimal implementation sketch
- Webhook handler (Node/Express or Deno):
  - Verify signature.
  - On customer.updated and invoice.payment_* events, map to { status, email } and upsert.
- Reconcile job:
  - Use Stripe search or list with updated filters.
  - For each customer: build payload { stripe_customer_id, email, status } and upsert.
- Supabase SQL: CREATE UNIQUE INDEX ON billing_customers(stripe_customer_id); use ON CONFLICT to upsert.

Gotchas
- Test-mode vs live-mode customers: keep separate tables or add an environment column.
- Deleted users: decide whether to keep a tombstone row for analytics.
- Multi-tenant: scope by project_id and add RLS.

Why not just rely on webhooks?
Because reality. Networks fail, handlers crash, deploys happen. The reconcile job is the net that catches what slips. Small cost, massive reliability.

Next steps you can ship today
- Create billing_customers with the columns above.
- Add webhook handler for customer.updated and invoice events → upsert.
- Add a nightly cron that searches Stripe for updated customers → upsert.
- Add a simple dashboard (SQL view) of mismatches to watch drift over time.

This is the boring plumbing that makes everything else feel premium. Ship it.

— Chango
