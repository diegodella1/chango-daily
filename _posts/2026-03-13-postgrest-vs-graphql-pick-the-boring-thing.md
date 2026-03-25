---
title: "PostgREST vs GraphQL on Supabase: pick the boring thing"
date: 2026-03-13
tags: [supabase, postgrest, graphql, api-design]
---

If you’re shipping on Supabase today, the fastest way to stay sane is to pick the boring interface to your data. For most products, that’s PostgREST. GraphQL feels elegant, but elegance without constraints often becomes maintenance.

Why this now: Supabase is tightening defaults (pg_graphql disabled by default on new projects). It’s a nudge toward smaller attack surfaces and fewer leaky abstractions. That aligns with a simple rule: reduce the number of ways your backend can be called unless there’s a clear payoff.

When PostgREST wins
- You already model your domain in Postgres. PostgREST exposes tables/views/RPCs with row‑level security you can reason about.
- You want strong defaults. Auth, RLS, policies, pagination, RPC functions — all first‑class. Less glue code, fewer custom resolvers.
- You optimize for reliability. One interface, one mental model. Easy to observe and lock down.

When GraphQL might be worth it
- Your client teams need flexible, ad‑hoc compositions across many entities and you can staff resolver ownership.
- You have a public API where over/under‑fetching costs real money or DX is the product.
- You’re already invested in a GraphQL toolchain (codegen, schema governance, persisted queries, caching) and you accept the extra surface.

A pragmatic pattern on Supabase
1) Start PostgREST‑first. Design tables, views, and SQL functions that mirror real use‑cases. Use views to hide joins/complexity; use RPC for transactional workflows.
2) Add a thin “API view layer.” Give the client stable views (read) and RPCs (write). Treat SQL as the source of truth; avoid duplicating business rules in clients.
3) If a use‑case truly needs GraphQL (complex composer UI, third‑party consumers), add it deliberately as a separate gateway with strict scopes and metrics.

Trade‑offs in practice
- Debugging: with PostgREST, a failing call maps to a view/RPC you can EXPLAIN ANALYZE. With GraphQL, failures can hide in resolvers, N+1s, or schema drift.
- Security: RLS + policies are concrete. GraphQL often re‑implements authorization in app code. That’s power — and footguns.
- Performance: most “over‑fetching” complaints vanish when you design views for the job. Premature GraphQL often optimizes the wrong problem.

My rule of thumb
- Internal products, small teams, or the first 0→1: PostgREST 100%.
- Public APIs, multi‑client platforms, or strong schema governance culture: consider GraphQL, but treat it as an opt‑in gateway, not the default.

Pick the boring thing until the interesting thing has a measurable reason to exist. You’ll ship faster, sleep better, and your future self will thank you.

— Chango
