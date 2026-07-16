# CLAUDE.md template

Write this into the new project's root as `CLAUDE.md` during Phase 3, filled in
with the project's real specifics. Claude Code loads it automatically every
session, so it is the cheapest way to keep future sessions consistent. Keep it
lean — a bloated CLAUDE.md wastes context every turn. Trim anything that stops
being true.

Copy the block below, replace the `<...>` parts, delete lines that don't apply.

```markdown
# <Project name>

## What this is
<One-line description and the single core user action.>

## Stack
Next.js (App Router) + TypeScript + Tailwind + Supabase + <Stripe if used> +
Vercel. Package manager: <npm/pnpm>.

## Commands
- Dev: `npm run dev`
- Build: `npm run build`
- Lint: `npm run lint`
- Test: `<test command, or "none yet">`

## Architecture rules
- Three Supabase clients live in `lib/supabase/`: `client.ts` (browser, anon),
  `server.ts` (server, per-user), `admin.ts` (service-role, SERVER-ONLY).
- Never import `admin.ts` or any non-`NEXT_PUBLIC_` env var into a client
  component.
- Every table holding user data has RLS enabled with owner-scoped policies.
  New tables are not "done" until their RLS policy exists and is tested.
- Stripe secret key and webhook handling are server-only. Webhooks verify the
  signature before trusting the event.

## Conventions
- Small, single-purpose commits with clear messages.
- `PROGRESS.md` at root holds current state: done, key decisions, next.
  Read it at session start; update it at every commit.
- Prefer server components; reach for client components only when needed.
- Match existing file and naming patterns in the repo before inventing new ones.
- Verify library APIs against current docs (Supabase auth, Stripe) rather than
  assuming — these change.

## Definition of done for a feature
Runs locally, verified in the browser (not just compiles), RLS/authz checked if
it touches data, committed. Note anything stubbed or skipped.

## Do NOT
- Do not paste secrets into tracked files or print full secrets.
- Do not add packages without confirming they exist on npm.
- Do not expand scope beyond the current spec — log new ideas under "Later".

## Later (parking lot)
<Running list of deferred ideas so they're captured without derailing v1.>
```

Tips:

- `/init` in Claude Code can generate a first draft by scanning the repo, but
  replace its generic output with the specifics above — the architecture and
  "Do NOT" rules are what actually prevent the expensive mistakes.
- Keep secrets and their values out of this file. It describes rules, not values.
- As the project grows, a subdirectory can get its own `CLAUDE.md` for local
  rules rather than bloating the root one.
