# SouthbankTowers — 100-Step Build Plan

> A production-grade resident portal for an apartment building.
> Complaints, an OC information portal, a social feed, feature flags, and SMS-verified accounts.

---

**Author** • Project owner & product lead
**Stack** • Next.js 16 · Supabase · Better Auth · shadcn/ui · Twilio Verify · Vercel
**Build tool** • Claude Code
**Estimated runtime** • 6–10 weeks part-time
**Last updated** • April 2026

---

## How to read this document

Each step lists three things: **Objective** (what you finish at the end), **Tech used** (the specific tool or library), and **Value** (why this step exists). Steps are ordered so each one only depends on what came before. Treat one step as one Claude Code session — sometimes shorter, occasionally a full afternoon.

---

## Phase 1 · Foundations (Steps 1–10)

### Step 1 — Decide the build philosophy
**Objective** Write a one-page README stating the project's mission, who it serves (residents + OC), and what success looks like in three months.
**Tech** Markdown, plain text.
**Value** Every later decision points back to this. Without it, scope creep wins.

### Step 2 — Lock the tech stack
**Objective** Commit to Next.js 16, Supabase, Better Auth, shadcn/ui, Twilio Verify, Vercel. Document why each was chosen alongside its runner-up.
**Tech** Architecture decision record (ADR) in `/docs/adr/0001-stack.md`.
**Value** Future you (and any contributor) will know why a piece exists, not just that it does.

### Step 3 — Create the GitHub repository
**Objective** Initialise a private repo named `southbank-towers` with a clean `main` branch, branch protection, and `.gitignore` for Node.
**Tech** GitHub, git CLI.
**Value** Source of truth and the trigger for every future deploy.

### Step 4 — Set up local dev environment
**Objective** Install Node 20 LTS, pnpm, and the Supabase CLI. Confirm versions in a `tooling.md`.
**Tech** Node, pnpm, Supabase CLI.
**Value** Eliminates "works on my machine" before it starts.

### Step 5 — Scaffold the Next.js app
**Objective** Run `pnpm create next-app` with TypeScript, App Router, Tailwind, ESLint. Verify the dev server boots.
**Tech** Next.js 16, TypeScript, Tailwind CSS.
**Value** A live, hot-reloading shell ready to receive every other piece of the system.

### Step 6 — Configure absolute imports and path aliases
**Objective** Set up `@/` to point to `src/`. Update `tsconfig.json` and confirm a sample import works.
**Tech** TypeScript path aliases.
**Value** Imports stay readable as the codebase grows past 50 files.

### Step 7 — Add Prettier and a shared editor config
**Objective** Install Prettier, add `.prettierrc`, `.editorconfig`, and a format-on-save VS Code setting.
**Tech** Prettier, EditorConfig.
**Value** Code style stops being a discussion. AI-generated code stays consistent.

### Step 8 — Author the master `CLAUDE.md`
**Objective** Write a project-level `CLAUDE.md` describing the stack, folder structure, naming conventions, design language (warm community feel), and rules for Claude Code (e.g. "use shadcn components, never raw HTML buttons").
**Tech** Markdown.
**Value** Claude Code reads this on every run. A good `CLAUDE.md` is the difference between a clean codebase and a sprawl.

### Step 9 — Define the colour and type tokens
**Objective** Lock the palette (terracotta, cream, sage, umber) and typography (Fraunces display, Inter Tight body) in `tailwind.config.ts` and a `globals.css` token layer.
**Tech** Tailwind CSS, Google Fonts.
**Value** A consistent identity. Every component built later inherits the look without thinking about it.

### Step 10 — Set up shadcn/ui
**Objective** Initialise shadcn with the New York style, the project's colour tokens, and install the first six primitives: Button, Input, Label, Card, Dialog, Toast.
**Tech** shadcn/ui CLI, Radix UI, Tailwind.
**Value** A baseline of accessible, customisable components. You own the source — fully styleable to the warm aesthetic.

---

## Phase 2 · Database & Backend Spine (Steps 11–20)

### Step 11 — Create the Supabase project
**Objective** Spin up a new Supabase project in the Sydney region. Save the URL, anon key, and service role key to a password manager.
**Tech** Supabase Cloud.
**Value** The data home for users, complaints, posts, and uploaded files. Sydney region keeps Australian residents' latency low and data local.

### Step 12 — Wire env vars into Next.js
**Objective** Create `.env.local` with the Supabase URL and anon key. Add `.env.example` to the repo, never the real `.env.local`.
**Tech** Next.js env handling, dotenv.
**Value** Secrets stay out of git. New contributors know what's needed.

### Step 13 — Install the Supabase client
**Objective** Add `@supabase/supabase-js` and create `src/lib/supabase/client.ts` (browser) and `src/lib/supabase/server.ts` (server).
**Tech** Supabase JS SDK.
**Value** A clean, single import path for every later database call.

### Step 14 — Design the data model on paper
**Objective** Sketch the tables: `users`, `units`, `complaints`, `complaint_comments`, `complaint_votes`, `info_posts`, `social_posts`, `social_comments`, `feature_flags`, `audit_log`. Note the relationships.
**Tech** dbdiagram.io or pen and paper.
**Value** Schema mistakes are 10× cheaper to fix on paper than after a migration.

### Step 15 — Write the initial schema migration
**Objective** Author `supabase/migrations/0001_init.sql` with the core tables, constraints, and indexes. Run it locally with `supabase db reset`.
**Tech** Supabase migrations, PostgreSQL.
**Value** Schema is now reproducible, version-controlled, and reviewable.

### Step 16 — Add Postgres ENUM types
**Objective** Define enums for `user_role` (resident, oc_member, admin), `complaint_status` (open, in_progress, resolved), `complaint_category` (noise, parking, maintenance, cleanliness, neighbour_dispute, security, other).
**Tech** PostgreSQL ENUM.
**Value** Invalid statuses become impossible at the database level — strongest possible constraint.

### Step 17 — Generate TypeScript types from the schema
**Objective** Run `supabase gen types typescript` and commit the result to `src/types/database.ts`.
**Tech** Supabase type generator.
**Value** Every query is now type-safe end-to-end. Renaming a column surfaces every affected file at compile time.

### Step 18 — Enable Row Level Security on every table
**Objective** Turn on RLS for all tables. Write `0002_rls.sql` with policies: residents read all complaints, only authors edit their own, OC can update status, anonymous complaints hide author from non-OC.
**Tech** Supabase RLS, PostgreSQL policies.
**Value** Security lives in the database, not application code. A buggy API route cannot leak data.

### Step 19 — Seed the database with realistic test data
**Objective** Write `supabase/seed.sql` creating 12 residents, 3 OC members, 8 sample complaints across categories, 4 info posts, and 6 social posts.
**Tech** SQL seed file.
**Value** Every developer (and Claude Code) can iterate against realistic data. UI screenshots look credible from day one.

### Step 20 — Set up Supabase Storage buckets
**Objective** Create three buckets: `complaint-photos` (public read, auth write), `info-documents` (auth read, OC write), `avatars` (public read, owner write). Add file size and MIME type policies.
**Tech** Supabase Storage.
**Value** A single integrated home for every file the app uploads. No second service to wire up.

---

## Phase 3 · Authentication with SMS Verification (Steps 21–32)

### Step 21 — Install Better Auth
**Objective** Add `better-auth`, configure its Postgres adapter pointed at Supabase, mount the handler at `/api/auth/[...all]`.
**Tech** Better Auth, Drizzle ORM (Better Auth's preferred driver).
**Value** A modern, self-hosted auth layer with no per-user vendor cost — and full ownership of the user table.

### Step 22 — Create the sign-up form
**Objective** Build `/sign-up` with fields for full name, email, password, apartment number, and mobile (E.164 format). Use shadcn `Form` + `react-hook-form` + `zod`.
**Tech** shadcn Form, react-hook-form, zod.
**Value** Every field validates client-side and server-side from the same schema — no contradictions.

### Step 23 — Create the sign-in form
**Objective** Build `/sign-in` with email + password. Add "forgot password" link and an inline error region.
**Tech** Better Auth `signIn.email`.
**Value** Returning residents have a single, predictable entry point.

### Step 24 — Add password reset flow
**Objective** Wire up Better Auth's reset endpoints, build the request and reset pages, send the link via Resend (configured in step 75).
**Tech** Better Auth password reset, Resend (later step).
**Value** Forgotten passwords don't become a support ticket to the OC.

### Step 25 — Sign up for Twilio and configure Verify
**Objective** Create a Twilio account, set up a Verify service, save the Account SID, Auth Token, and Verify Service SID to env vars.
**Tech** Twilio Verify.
**Value** OTPs are a managed service — no DIY code generation, no SMS deliverability headaches.

### Step 26 — Build the SMS OTP send endpoint
**Objective** Create `/api/auth/sms/send` that receives a phone number, calls Twilio Verify's `/verifications` endpoint, returns success.
**Tech** Twilio Verify REST API, Next.js Route Handler.
**Value** A single, server-side call point — Twilio credentials never touch the browser.

### Step 27 — Build the SMS OTP verify endpoint
**Objective** Create `/api/auth/sms/verify` that takes phone + code, calls Twilio's `/verificationCheck`, marks the user's `phone_verified` flag in Postgres on success.
**Tech** Twilio Verify check API, Supabase update.
**Value** Phone ownership is now provably linked to the account — the foundation for any future SMS feature.

### Step 28 — Add the SMS step to the sign-up flow
**Objective** After email/password registration, route the user to `/verify-phone`. Show a 6-digit OTP input, resend button (60-second cooldown), and a success state.
**Tech** shadcn `InputOTP`, React state.
**Value** Every account in the system is tied to a real phone — drastically reduces fake accounts and spam.

### Step 29 — Build the role-promotion logic
**Objective** Add a Postgres function `promote_to_oc(user_id)` callable only by existing OC members. Surface it later in the admin UI.
**Tech** Postgres functions, RLS.
**Value** OC membership becomes a controlled, auditable transition — not something an attacker can self-grant.

### Step 30 — Create the auth middleware
**Objective** Add `middleware.ts` that protects `/dashboard/*`, `/complaints/*`, `/info/*`, `/social/*`. Unauthenticated users go to `/sign-in?next=…`.
**Tech** Next.js middleware, Better Auth `getSession`.
**Value** A single chokepoint. No protected page can accidentally render to a logged-out user.

### Step 31 — Build a `useUser` hook and `useRequireRole` guard
**Objective** Wrap session access in a typed hook. Add a guard component that 403s residents from OC-only routes.
**Tech** React, Better Auth client.
**Value** Authorisation logic is reusable, testable, and consistent everywhere.

### Step 32 — Write the auth integration tests
**Objective** Use Playwright to write tests covering: sign-up → SMS verify → sign-in → protected route access → sign-out.
**Tech** Playwright.
**Value** The most security-critical path in the app is now regression-protected.

---

## Phase 4 · Application Shell & Navigation (Steps 33–40)

### Step 33 — Design the marketing landing page
**Objective** Build `/` with a hero ("A building is its people"), three feature cards (Complaints, Notices, Community), and a sign-in CTA. Use the warm community palette.
**Tech** Next.js, Tailwind, shadcn.
**Value** First impression for new residents and a portfolio-ready public face.

### Step 34 — Build the authenticated app shell
**Objective** Create `(app)` route group with a left sidebar (Home, Complaints, Info, Social, Profile), a top bar with notification bell, and a main content area.
**Tech** Next.js layouts, route groups.
**Value** Every authenticated page inherits navigation for free. One place to update, not seven.

### Step 35 — Build the dashboard home
**Objective** `/dashboard` shows: latest 3 announcements, complaint count by status, latest 3 social posts, "Submit complaint" CTA.
**Tech** Server Components, Supabase queries.
**Value** Residents see what matters within five seconds of logging in.

### Step 36 — Add a global toast system
**Objective** Wire up shadcn Toast and a `useToast` helper. Confirm every form mutation can fire success and error toasts.
**Tech** shadcn Toast, Radix.
**Value** User feedback is consistent across every action — no jarring surprises.

### Step 37 — Build the profile page
**Objective** `/profile` lets the user update name, avatar (uploaded to Supabase Storage), email (re-verification required), and phone (re-OTP required).
**Tech** Supabase Storage upload, image cropping (`react-easy-crop`).
**Value** Residents own their identity in the system without going through the OC.

### Step 38 — Add a 404 and a custom error page
**Objective** Build `not-found.tsx` and `error.tsx` with on-brand illustrations and a path back to safety.
**Tech** Next.js error boundaries.
**Value** Errors stop being scary blank screens — they become moments of trust.

### Step 39 — Implement the loading skeleton system
**Objective** Add `loading.tsx` files for the three main routes. Use shadcn `Skeleton` to mirror the eventual layout.
**Tech** Next.js streaming, shadcn Skeleton.
**Value** Perceived performance jumps. Residents see structure immediately while data fetches.

### Step 40 — Add an "About this building" page
**Objective** `/about` with a short story of Southbank Towers, photos, OC introduction. Editable later by OC.
**Tech** Static MDX.
**Value** Community is built by story, not data tables. This is where new residents learn what they belong to.

---

## Phase 5 · Complaints Module (Steps 41–55)

### Step 41 — Build the complaints list page
**Objective** `/complaints` lists all complaints, newest first, with status badges, category, vote count, and comment count.
**Tech** Server Component, Supabase query.
**Value** Residents can see at a glance what's being raised in their building.

### Step 42 — Add complaint filtering
**Objective** Add tabs for All / Open / In progress / Resolved, plus a category dropdown.
**Tech** Next.js search params, shadcn Tabs.
**Value** A 200-complaint list stays scannable. URLs are shareable.

### Step 43 — Build the "New complaint" form
**Objective** Modal with title, description, category dropdown, "Submit anonymously" toggle, photo uploader (max 4 images, 5 MB each).
**Tech** shadcn Dialog, react-dropzone, Supabase Storage.
**Value** The single most important user action in the app — getting it right matters.

### Step 44 — Implement photo upload to Supabase Storage
**Objective** Upload selected images to `complaint-photos` bucket, return signed URLs, store URLs in `complaints.photos[]`.
**Tech** Supabase Storage, signed URLs.
**Value** Evidence makes complaints actionable — a noisy neighbour photo is worth a paragraph.

### Step 45 — Build the complaint detail page
**Objective** `/complaints/[id]` shows full description, photo carousel, timeline of status changes, vote button, comment thread.
**Tech** Dynamic routes, Server Components.
**Value** A complaint becomes a conversation, not a one-shot ticket.

### Step 46 — Add the upvote system
**Objective** Wire the vote button to a `complaint_votes` table. One vote per user per complaint, optimistic UI update.
**Tech** Supabase upsert, React optimistic updates.
**Value** Genuine community concerns rise to the top — OC sees priorities, not just timestamps.

### Step 47 — Build the comment thread
**Objective** Threaded comments on each complaint. Residents can comment, OC comments get a green "OC" badge.
**Tech** Supabase joins, role-aware rendering.
**Value** Discussion happens in context — no parallel WhatsApp groups required.

### Step 48 — Implement OC status updates
**Objective** OC members see an "Update status" dropdown on the complaint detail page. Changes write to a `complaint_status_log` table.
**Tech** Role-gated UI, audit log.
**Value** Residents can trust the status — every change is timestamped and attributed.

### Step 49 — Add anonymous complaint masking
**Objective** When `is_anonymous = true`, the author's name is replaced with "Resident" everywhere except for OC, who can see the real author behind a "Reveal" button (logged).
**Tech** RLS, conditional joins.
**Value** Sensitive complaints (neighbour disputes, harassment) get raised without fear, while keeping accountability for OC.

### Step 50 — Build the OC complaints dashboard
**Objective** `/oc/complaints` for OC members only — a sortable table with bulk status change, average resolution time, and complaint volume by week.
**Tech** TanStack Table, Recharts.
**Value** OC stops drowning in individual tickets and starts seeing the system.

### Step 51 — Add complaint search
**Objective** Add a search box on `/complaints` that searches title and description using Postgres full-text search.
**Tech** Postgres `tsvector`, GIN index.
**Value** "Has anyone else complained about the lift?" gets answered in two seconds.

### Step 52 — Implement complaint pagination
**Objective** Server-side cursor pagination, 20 per page. Add a "Load more" button.
**Tech** Supabase range queries, cursor pagination.
**Value** Performance stays flat as the building accumulates complaints over years.

### Step 53 — Add real-time complaint updates
**Objective** Subscribe to Supabase `complaints` channel — new complaints and status changes appear without refresh.
**Tech** Supabase Realtime.
**Value** The app feels alive. Residents trust that what they see is current.

### Step 54 — Add complaint export for OC
**Objective** OC can download a CSV of complaints filtered by date range, category, or status.
**Tech** `papaparse`, client-side download.
**Value** AGM reports and quarterly reviews stop being weekend work.

### Step 55 — Write tests for the complaints flow
**Objective** Playwright tests: submit complaint → upvote → comment → OC status change → resolution.
**Tech** Playwright.
**Value** The most-used module is regression-proof going forward.

---

## Phase 6 · OC Information Portal (Steps 56–65)

### Step 56 — Build the info portal home
**Objective** `/info` shows pinned announcements, recent posts, and a category filter (Announcements, Meeting Minutes, By-laws, Maintenance, Emergency Contacts, Financial Reports).
**Tech** Server Components.
**Value** A single canonical source for "what's happening in the building" — replacing notice boards and lost emails.

### Step 57 — Build the OC post composer
**Objective** OC-only modal: title, rich-text body (Tiptap), category, optional PDF attachment, optional pin-to-top, schedule-for-later option.
**Tech** Tiptap editor, Supabase Storage.
**Value** OC communications look professional. PDFs (meeting minutes, financial reports) live alongside the announcement, not in someone's inbox.

### Step 58 — Implement PDF attachment upload
**Objective** Drag-and-drop PDF upload to `info-documents` bucket, max 10 MB, virus scan via Supabase function.
**Tech** Supabase Storage, ClamAV edge function (optional).
**Value** Documents are safe to download — no malware concerns from residents.

### Step 59 — Build the post detail page
**Objective** `/info/[id]` shows formatted post, attached PDFs as inline previews, "posted by" attribution, optional comment thread (toggle by OC).
**Tech** PDF.js for inline preview.
**Value** Information is consumed where it's posted — no friction, no "where do I find it" later.

### Step 60 — Add post pinning
**Objective** OC can pin up to 3 posts to the top. Pinned posts have a distinct visual treatment.
**Tech** Boolean `is_pinned`, sort priority.
**Value** Critical announcements (lift outage, water shutdown) stay visible without spam.

### Step 61 — Implement post scheduling
**Objective** Posts can be saved with a `publish_at` timestamp. A daily cron promotes them at the right time.
**Tech** Supabase scheduled functions.
**Value** OC can prepare announcements during the week and have them go out on Friday morning.

### Step 62 — Add post versioning
**Objective** Edits to an info post create a new row in `info_post_revisions`. Show "edited" with a hover-to-see-diff.
**Tech** Trigger-based history table.
**Value** Trust. Residents can verify nothing was quietly changed after a controversy.

### Step 63 — Build the maintenance schedule view
**Objective** A dedicated `/info/maintenance` page rendering a calendar of scheduled events (lift servicing, fire alarm tests, garden work).
**Tech** `react-big-calendar` or FullCalendar.
**Value** Residents stop being surprised when the lift is out for a day.

### Step 64 — Build the emergency contacts directory
**Objective** A static-feeling page (data in DB, OC-editable) listing building manager, after-hours plumber, electrician, locksmith, security.
**Tech** Simple table view.
**Value** A 2 a.m. burst pipe doesn't require digging through emails.

### Step 65 — Add an info post search
**Objective** Full-text search across all posts and PDF text (extracted on upload via `pdf-parse`).
**Tech** Postgres FTS, `pdf-parse`.
**Value** Two years of meeting minutes become searchable, not archaeological.

---

## Phase 7 · Social Feed (Steps 66–73)

### Step 66 — Build the social feed page
**Objective** `/social` is a vertical feed, newest first, showing post text, optional image, author, timestamp, like count, comment count.
**Tech** Server Component, infinite scroll.
**Value** A casual space to build neighbourliness — "anyone got a spare drill?" or "lost cat in lobby."

### Step 67 — Build the post composer
**Objective** Inline composer at the top: text area, single optional image, post button.
**Tech** Auto-resize textarea, image upload.
**Value** Frictionless posting. The lower the bar, the more community emerges.

### Step 68 — Add likes (heart reactions)
**Objective** One-tap heart reaction. Counts update optimistically.
**Tech** Supabase upsert, React optimistic state.
**Value** Lightweight encouragement keeps positive activity flowing.

### Step 69 — Build the comment thread for social posts
**Objective** Inline expandable comment thread per post. Limit nesting to one level for simplicity.
**Tech** Recursive rendering, soft limit.
**Value** Conversation without the chaos of unbounded nesting.

### Step 70 — Add OC moderation tools
**Objective** OC can hide a post (soft delete with reason), and any user can report a post. Reports appear in `/oc/moderation`.
**Tech** Soft delete column, report table.
**Value** The community stays welcoming without heavy-handed pre-moderation.

### Step 71 — Implement basic content rules
**Objective** Auto-flag posts containing personal details (mobile numbers, full addresses) using regex pre-checks. Flagged posts go to OC review.
**Tech** Server-side regex, async queue.
**Value** Privacy protection without making honest posts feel surveilled.

### Step 72 — Add real-time feed updates
**Objective** New posts and comments stream in via Supabase Realtime — a "3 new posts" pill appears at the top.
**Tech** Supabase Realtime channels.
**Value** The feed feels alive when the building is active (think Friday evening barbecue chatter).

### Step 73 — Build the user mini-profile popover
**Objective** Hover (or tap on mobile) over an author's name shows apartment number, joined date, and a "Send a hello" button (creates a new social post tagged @them).
**Tech** Radix Popover.
**Value** Real names attached to real apartments — the social glue of an actual community.

---

## Phase 8 · Notifications (Steps 74–80)

### Step 74 — Design the notifications data model
**Objective** Add `notifications` table with `user_id`, `type`, `payload` (jsonb), `read_at`, `created_at`. Index on `(user_id, created_at desc)`.
**Tech** Postgres jsonb.
**Value** Polymorphic notifications without a table-per-type explosion.

### Step 75 — Set up Resend for transactional email
**Objective** Sign up for Resend, verify a sending domain, store API key. Build a `sendEmail` helper.
**Tech** Resend.
**Value** Password resets, OC invitations, and weekly digests now have a delivery path.

### Step 76 — Build the in-app notification dropdown
**Objective** Bell icon in the top bar shows unread count. Dropdown lists last 20 notifications, click marks read.
**Tech** Server Component, Realtime subscription.
**Value** The primary delivery surface — and what residents asked for.

### Step 77 — Implement notification triggers
**Objective** Postgres triggers create notification rows on: new comment on your complaint, status change, new info post, like on your social post.
**Tech** Postgres triggers, Supabase functions.
**Value** Logic lives in the database — impossible for an API route to forget to notify.

### Step 78 — Add notification preferences
**Objective** Profile page lets each user toggle which notification types they receive.
**Tech** `notification_preferences` table.
**Value** Residents control their experience — no opt-out fatigue.

### Step 79 — Add real-time notification delivery
**Objective** Subscribe to the user's notification channel. The bell badge updates without refresh.
**Tech** Supabase Realtime.
**Value** Notifications feel instant — not "next time you reload."

### Step 80 — Build a weekly email digest (opt-in)
**Objective** Sunday-evening email summarising the week: new complaints, OC posts, top social posts. Opt-in toggle in profile.
**Tech** Supabase scheduled function, Resend, MJML templates.
**Value** Residents who don't open the app stay connected. A gentle, low-pressure cadence.

---

## Phase 9 · Feature Management (Steps 81–86)

### Step 81 — Design the feature flag system
**Objective** Add `feature_flags` table: `key`, `description`, `enabled`, `rollout_percentage`, `target_roles[]`. Build a `useFlag('key')` hook.
**Tech** Postgres, React context.
**Value** Risky features can ship dark, then roll out gradually — and roll back instantly without a deploy.

### Step 82 — Build the flag admin UI
**Objective** `/admin/flags` for OC chair only. Table with toggles, percentages, role targeting, and last-changed-by.
**Tech** TanStack Table, audit log.
**Value** Non-engineers (the OC chair) control feature exposure without filing tickets.

### Step 83 — Wire the first three flags into the app
**Objective** Flag the social feed, the weekly digest, and the OC moderation queue. Confirm the UI vanishes and reappears on toggle.
**Tech** `useFlag`, conditional rendering.
**Value** Proof the system works end-to-end. Each future feature gets a flag for free.

### Step 84 — Add A/B variant support
**Objective** Extend flags with `variants` (e.g. `{control, blue_button, green_button}`) and a deterministic user-bucket function.
**Tech** Hash-based bucketing.
**Value** Future-proofs the system for measuring whether a redesign actually helps.

### Step 85 — Add a flag-evaluation audit log
**Objective** Every flag check (sampled) writes to an aggregate table. Admin sees: flag X evaluated 1,200 times in 24 hours, 60% true.
**Tech** Sampled logging, aggregation.
**Value** OC can see whether a flag is even being hit — debugging "why isn't this working" becomes trivial.

### Step 86 — Document flag lifecycle
**Objective** Add a `FLAGS.md` listing every flag, its purpose, owner, and removal date. Old flags get pruned in a quarterly review.
**Tech** Markdown.
**Value** Stops the codebase from accumulating dead flags — the most common feature-flag failure mode.

---

## Phase 10 · Hardening, Polish, Launch (Steps 87–100)

### Step 87 — Add observability with Sentry
**Objective** Install Sentry for both client and server. Confirm a deliberate test error surfaces in the dashboard.
**Tech** Sentry.
**Value** When something breaks at 11 p.m., you see it before residents do.

### Step 88 — Add basic analytics with PostHog
**Objective** Wire PostHog for page views and key events (complaint submitted, post created, OC promoted). Self-hosted or EU cloud for privacy.
**Tech** PostHog.
**Value** Decisions about what to build next get based on data, not guesswork.

### Step 89 — Run an accessibility audit
**Objective** Run axe DevTools against every major page. Fix every critical and serious issue. Confirm keyboard navigation works end-to-end.
**Tech** axe DevTools, manual keyboard testing.
**Value** Accessibility isn't optional — it's a legal expectation, and it makes the app better for everyone.

### Step 90 — Add structured rate limiting
**Objective** Use Upstash Redis to rate-limit auth endpoints (5/min), complaint submissions (3/min), and SMS sends (1/min).
**Tech** Upstash Redis, `@upstash/ratelimit`.
**Value** Spam, scraping, and SMS-cost-attacks are mitigated before they happen.

### Step 91 — Write an end-to-end smoke test suite
**Objective** Playwright suite covering: full sign-up → complaint → comment → social post → notification — runs on every CI pipeline.
**Tech** Playwright, GitHub Actions.
**Value** Confidence that a routine deploy hasn't broken the critical path.

### Step 92 — Set up CI/CD
**Objective** GitHub Actions workflow: lint, type-check, unit tests, e2e tests, then deploy to Vercel preview on PR and production on main.
**Tech** GitHub Actions, Vercel.
**Value** Every change is verified the same way, every time.

### Step 93 — Configure database backups and point-in-time recovery
**Objective** Confirm Supabase daily backups are running. Test a restore-to-point-in-time on a clone project.
**Tech** Supabase backups.
**Value** A rogue migration or accidental delete is recoverable in minutes, not "everything's gone."

### Step 94 — Write a privacy policy and terms
**Objective** Author plain-English `/privacy` and `/terms` pages. Cover what data is collected, how SMS verification works, and the OC's data access.
**Tech** Markdown, MDX.
**Value** Legal compliance (Australian Privacy Principles), and — more importantly — informed residents.

### Step 95 — Run a security review
**Objective** Manually verify: RLS on every table, no service-role key in client code, secure cookie flags, CSP header, dependency vulnerabilities (`pnpm audit`).
**Tech** Manual review, OWASP checklist.
**Value** A building-management app holds personal data — it deserves the same rigour as a small bank.

### Step 96 — Onboard the first three OC members
**Objective** Create their accounts manually, walk them through the OC dashboard, gather first feedback in a shared Notion doc.
**Tech** Notion or a simple Google Doc.
**Value** OC champions become ambassadors. They will sell the app to other residents.

### Step 97 — Soft-launch to 10 friendly residents
**Objective** Invite 10 residents you trust, give them a one-page quickstart, and ask for one specific piece of feedback after a week.
**Tech** Email invitation, Tally feedback form.
**Value** Real-world feedback before the full building sees it. Bugs that survive ten users are usually the worst kind.

### Step 98 — Iterate based on the first week's feedback
**Objective** Triage feedback into a public roadmap (Trello / Linear). Ship the top 5 fixes within a week.
**Tech** Linear or Trello.
**Value** Residents see their feedback acted upon — the strongest possible trust-builder.

### Step 99 — Hold the full-building launch
**Objective** OC sends a building-wide announcement. Print and post a one-page poster in lifts and lobbies with the URL and a QR code.
**Tech** Canva poster, building noticeboards.
**Value** Critical mass — the social feed and complaint upvoting only work with a real number of people on the platform.

### Step 100 — Write the post-launch retrospective
**Objective** A `RETROSPECTIVE.md` covering what shipped, what didn't, what surprised you, and the next quarter's priorities. Pin it in the repo README.
**Tech** Markdown.
**Value** The project becomes portfolio-ready — not just a working app, but a documented journey. This is the artefact that wins interviews, OC gratitude, and future projects.

---

## Appendix · The whole stack at a glance

| Layer | Tool |
|---|---|
| Framework | Next.js 16 (App Router, TypeScript) |
| UI | shadcn/ui · Tailwind CSS · Fraunces · Inter Tight |
| Auth | Better Auth |
| SMS | Twilio Verify |
| Database | Supabase (Postgres) |
| Storage | Supabase Storage |
| Realtime | Supabase Realtime |
| Email | Resend |
| Rate limiting | Upstash Redis |
| Hosting | Vercel |
| Monitoring | Sentry |
| Analytics | PostHog |
| Testing | Playwright |
| CI/CD | GitHub Actions |

## Appendix · Suggested cadence

If you build for ~10 hours a week, a realistic cadence is one phase every 7–10 days, finishing the full plan in 8–10 weeks. The first three phases are the slowest; once auth and the data model are in, every later module clones the same pattern.

## Appendix · Portfolio framing

When this is done, the README should lead with three things: a 30-second video walkthrough, a one-paragraph problem statement (the people in your building, the OC's pain, the residents' silence), and a link to this build plan. The plan itself is the second-strongest portfolio artefact — it shows not just that you can build, but that you can think.
