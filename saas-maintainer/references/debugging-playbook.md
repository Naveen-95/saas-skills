# Debugging playbook

Reproduce first, then read, then fix the cause. Most wasted debugging time comes
from changing code before understanding what's actually happening.

## The method

1. **Reproduce.** Exact steps, full error text, expected vs actual. No repro, no
   verifiable fix.
2. **Read the path.** Follow the actual code the failing input runs through.
   Read it and its neighbors before theorizing.
3. **Form one hypothesis.** State the suspected cause in a sentence. Resist
   shotgun-changing several things at once — you won't know which one mattered.
4. **Smallest change that tests the hypothesis.** If it fixes it, good. If not,
   revert it and form a new hypothesis. Don't pile changes on top of failed
   guesses.
5. **Verify + regression-check.** Repro steps pass, and nothing neighboring
   broke.

If you find yourself adding code to *hide* the symptom (swallowing an error,
defaulting an empty result to something, silencing a warning), stop — that's a
sign you haven't found the cause yet.

## The two-strikes rule (fix-loop escape)

If the same fix approach has failed **twice**, do not try it a third time.
Repeated retries in a long session pollute context — earlier failed attempts
crowd the window and the model starts guessing. The escape is a hard reset:

1. **Stop.** No third retry of the same approach.
2. **Revert** to the last working commit so the code is clean.
3. **Fresh context** (`/clear` or a new session). Restate in 3 lines only:
   the goal (one sentence), the exact sticking point (one or two sentences),
   and the last working state. Do NOT carry over the failed history — that is
   the poison, not the cure.
4. In the fresh context: list the 3 most likely causes simplest-first, fix
   ONLY the most likely one, state exactly how to confirm it worked, and
   explicitly avoid the approach that already failed twice.

---

## Stack failure signatures — what looks like a bug but isn't

These are the traps on this stack. When a symptom matches, look here before
"fixing" the wrong layer.

### Supabase

- **Empty result / "no rows" when data clearly exists** → very often RLS working
  correctly, not a bug. The current user simply isn't allowed to see those rows,
  or the query runs with the anon client where a per-user server client was
  needed. Check the policy and which client is being used **before** touching
  data. Do NOT "fix" this by disabling RLS.
- **Works locally, fails in production (or vice versa)** → usually an env var
  missing in one environment, or local using service-role while prod uses anon.
- **Auth/session is null in a server component or route handler** → cookie/session
  handling for the App Router, not a login bug. Confirm the current
  `@supabase/ssr` server-client cookie pattern.
- **Writes silently do nothing** → an RLS insert/update policy is missing (a
  policy for select doesn't cover insert).

### Next.js (App Router)

- **Stale data that won't update** → caching, not a data bug. `fetch` caching and
  route segment config (`revalidate`, `dynamic`) can serve old values. Look at
  cache settings before suspecting the database.
- **"useState/useEffect is not defined" or hook errors** → a server component
  using client-only features. Check the `'use client'` boundary.
- **Hydration mismatch warnings** → server and client rendered different HTML,
  often from using `Date`/`Math.random`/`window` during render. Not a data bug.
- **`process.env.X` is undefined in the browser** → it lacks the `NEXT_PUBLIC_`
  prefix. Do NOT fix this by making a secret `NEXT_PUBLIC_` — if it's secret, the
  code using it belongs on the server.

### Stripe

- **Webhook never fires locally** → you need `stripe listen` forwarding to your
  local endpoint; the event isn't reaching you. Not a code bug.
- **Webhook returns signature verification failed** → the handler is parsing the
  body before verifying, or using the wrong webhook secret. Signature
  verification needs the raw request body. Fix the body handling — never fix it
  by skipping verification.
- **Payment "succeeds" but the user gets nothing** → entitlement is being granted
  on the client success redirect instead of from the verified webhook, and the
  webhook path is broken or missing.
- **Nothing works and keys look right** → test-mode vs live-mode key mismatch
  (test key against live data or vice versa).

### General

- **Type error after an edit** → read what the type actually says; it's usually
  pointing at a real mismatch, not noise. Don't `any`-cast it away.
- **"It broke after I added a package"** → check the dependency's breaking
  changes; consider whether the package is even real (hallucinated/squatted
  packages exist).

---

## When you genuinely can't reproduce

- Ask for the exact input, the user's role/account state, timestamp, and full
  error/stack.
- Check logs (Vercel, Supabase, Sentry if connected) for the actual failure
  rather than guessing.
- If it's intermittent, suspect state that differs between requests: caching,
  auth/session, race conditions, environment differences.
- Do not ship a speculative fix for a bug you never reproduced and call it done
  — say clearly that it's a hypothesis and how the user can confirm.
