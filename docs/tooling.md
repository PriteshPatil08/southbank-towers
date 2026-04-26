# Tooling

Confirmed versions as of April 2026. New contributors should match these exactly to avoid environment-specific bugs.

| Tool | Version | Install |
|---|---|---|
| Node.js | v24.14.1 | https://nodejs.org (use the LTS installer or `nvm`) |
| pnpm | 10.33.2 | `npm install -g pnpm` |
| npm | 11.11.0 | Bundled with Node |
| Supabase CLI | 2.95.3 | `npm install -g supabase` or via `npx supabase` |

## Notes

- **Node version:** The build plan specified Node 20 LTS; v24.14.1 (current active release) is used instead and is fully compatible. If you need to manage multiple Node versions, use `nvm` (Mac/Linux) or `nvm-windows`.
- **pnpm over npm:** All install and script commands in this project use `pnpm`. Do not use `npm install` — it will generate a `package-lock.json` alongside `pnpm-lock.yaml` and cause conflicts.
- **Supabase CLI:** Used for local development (`supabase start`), running migrations (`supabase db reset`), and generating TypeScript types (`supabase gen types typescript`). Can be run via `npx supabase` without a global install, but a global install is recommended.

## Verify your environment

Run this to confirm everything is in order:

```bash
node --version    # should be v24.x or v20.x LTS
pnpm --version    # should be 10.x
supabase --version # should be 2.x
```
