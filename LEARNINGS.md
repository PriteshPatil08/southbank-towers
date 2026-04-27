# Learnings

## Step 1 — Decide the build philosophy

> A README written before a single line of code is a forcing function — it makes you articulate who you're building for and what "done" actually means.
> Success criteria without numbers are just wishes. "Residents trust the app" is unmeasurable. "Zero privacy complaints in three months" is not.
> The two audiences (residents and OC) have different pain, different permissions, and different definitions of value — designing for one without the other breaks the system.

**Technical Topics**
- Markdown authoring
- Product requirement framing
- Audience segmentation
- Defining measurable success criteria

---

## Step 2 — Lock the tech stack

> An Architecture Decision Record is not documentation — it's a binding contract with your future self that explains why the bridge was built this way.
> The runner-up matters as much as the winner: naming what lost and why proves the decision was deliberate, not accidental.
> Consequences are the most honest section of any ADR — what you commit to, what you rule out, and what bets you're knowingly making.

**Technical Topics**
- Architecture Decision Records (ADRs)
- Next.js 16 App Router
- Supabase (Postgres, RLS, Storage, Realtime)
- Better Auth
- shadcn/ui component model
- Twilio Verify
- Vercel deployment model

---

## Step 3 — Create the GitHub repository

> A `.gitignore` is not housekeeping — it's the last line of defence between your secrets and the public internet.
> Branch protection turns `main` from a convention into a constraint: force pushes and deletions become physically impossible, not just frowned upon.
> Renaming `master` to `main` requires changing the default branch on GitHub before deleting the old remote ref — order matters.

**Technical Topics**
- Git branch renaming and remote tracking
- GitHub default branch configuration
- `.gitignore` patterns for Node/Next.js projects
- GitHub branch protection rules via the API
- `gh` CLI repo management

---

## Step 4 — Set up local dev environment

> Documenting tool versions is not bureaucracy — it's the diff between "works on my machine" and "works on every machine."
> pnpm is not just a faster npm; it uses a content-addressable store that prevents phantom dependency bugs that npm's hoisting model allows.
> The Supabase CLI is the local control plane for the database — migrations, type generation, and local dev all run through it.

**Technical Topics**
- Node.js version management (nvm)
- pnpm vs npm: hoisting, lockfiles, and the node_modules structure
- Supabase CLI: local dev, migrations, type generation
- Environment reproducibility

---

## Step 5 — Scaffold the Next.js app

> `create next-app` is opinionated by design — each flag (`--app`, `--src-dir`, `--typescript`) locks in an architectural choice so you never have to revisit it.
> pnpm uses hard links and symlinks in node_modules; a plain `cp` breaks them — always reinstall with `pnpm install` rather than copying node_modules between directories.
> Next.js 16 with Turbopack boots in under a second; the old Webpack-based dev server took 10–30 seconds on cold start.

**Technical Topics**
- Next.js 16 App Router scaffolding
- Turbopack (Next.js's Rust-based bundler, replacing Webpack in dev)
- pnpm's content-addressable store and symlink structure
- TypeScript + Tailwind CSS 4 integration
- ESLint 9 flat config format (`eslint.config.mjs`)
