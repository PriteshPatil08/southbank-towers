# ADR 0001 — Technology Stack

**Status:** Accepted
**Date:** April 2026
**Deciders:** Project owner

---

## Context

Southbank Towers is a resident portal for an apartment building. It needs accounts with SMS verification, a complaints tracker, an OC information portal, a social feed, and a feature flag system. It will be built and maintained by one developer, needs to launch in under 10 weeks part-time, and must handle Australian residents' data without sending it offshore.

The stack must:
- minimise operational overhead (no self-hosted infra)
- keep costs near zero until the building actually uses it
- be composable enough to add modules without rewriting the core
- be modern enough that AI tooling (Claude Code) can assist effectively

---

## Decisions

### Framework — Next.js 16 (App Router)
**Runner-up:** Remix

Next.js 16 with the App Router gives us React Server Components, file-based routing, and built-in API route handlers — meaning the frontend and the backend live in one repo, one deploy, one mental model. Remix was a serious contender (excellent data loading patterns, better progressive enhancement), but Next.js has a larger ecosystem, better shadcn/ui integration, and Vercel's first-party deploy pipeline removes an entire class of configuration problems.

### Database & Backend — Supabase
**Runner-up:** PlanetScale + custom API

Supabase gives us Postgres, authentication primitives, row-level security, file storage, realtime subscriptions, and edge functions — all in one dashboard, all in the Sydney region. PlanetScale is excellent but MySQL-only (no RLS, no PostGIS), and pairing it with a custom API layer would triple the build time. Supabase's free tier handles a building's worth of traffic comfortably; scaling is a flip of a switch.

### Authentication — Better Auth
**Runner-up:** Supabase Auth

Better Auth is a self-hosted, framework-agnostic auth library with first-class Next.js support, a Postgres adapter, and no per-user pricing. Supabase Auth was the obvious default — it's already in the stack — but it imposes its own user table schema and makes customising the sign-up flow (e.g. storing apartment number at registration) awkward. Better Auth owns the user table completely, which gives us the flexibility needed for the SMS verification flow and OC role promotion.

### UI Components — shadcn/ui
**Runner-up:** Mantine

shadcn/ui is not a component library — it's a code generator. You own every component it installs; there is no upstream version to pin. That means full Tailwind theming, zero bundle bloat from unused components, and no breaking changes from a maintainer's redesign decision. Mantine is more fully-featured out of the box, but its theming system fights Tailwind. For a project with a strong visual identity (terracotta, cream, sage), owning the source is non-negotiable.

### SMS Verification — Twilio Verify
**Runner-up:** MessageBird (now Bird)

Twilio Verify is a managed OTP service — no DIY code generation, no expiry logic, no retry handling. It handles all of that, plus delivery receipts and fraud detection. MessageBird offers comparable features at slightly lower cost, but Twilio's Australian carrier relationships mean better delivery rates on local numbers, and its documentation is significantly better for a first-time integration.

### Hosting — Vercel
**Runner-up:** Fly.io

Vercel is the first-party host for Next.js — preview deployments on every PR, automatic production deploys on merge to main, edge network, and zero configuration. Fly.io would give more control over the runtime environment, but "more control" is a cost in a one-developer project. Vercel's free tier covers the build and traffic comfortably at building scale.

---

## Consequences

**What this stack commits us to:**
- Postgres as the single source of truth. All business logic that can live in the database (RLS, triggers, functions) should — this is the core Supabase design contract.
- TypeScript end-to-end. Supabase type generation and Better Auth both produce typed interfaces; diverging to plain JS anywhere creates a gap that will bite us.
- The App Router mental model. Server Components, `use client` boundaries, and `loading.tsx` / `error.tsx` conventions are not optional — they're how the framework works.
- Vercel for deploys. Migrating away later is possible but not free; accept this as the hosting primitive for the project's lifetime.

**What this stack rules out:**
- MySQL, MongoDB, or any non-Postgres datastore — Supabase RLS and Better Auth's adapter are Postgres-only.
- Hosting outside Australia (for primary data) — Supabase Sydney region is a deliberate privacy decision; don't add a secondary region that stores user data offshore.
- Client-side Supabase service role access — the service role key bypasses RLS entirely. It must never touch the browser. Ever.

**Known risks:**
- Better Auth is younger than Supabase Auth or Auth.js. Its API may shift between minor versions during the build. Pin the version and read the changelog before upgrading.
- Twilio Verify costs money per verification (~AU$0.10). At building scale this is trivial; if the app ever expands beyond one building, revisit.
