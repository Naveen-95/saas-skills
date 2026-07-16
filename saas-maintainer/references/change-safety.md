# Change safety

Rules for changing an app that has real users and real data. The theme: changes
should be small, reversible, and never trade away safety for convenience.

---

## Database migrations on live data

A migration that's harmless on an empty dev database can destroy real user data
in production. Treat every migration against live data as high-stakes.

**Safe by default (additive, reversible):**
- Adding a new table.
- Adding a **nullable** column, or a column with a default.
- Adding an index (mind lock behavior on large tables).

**Dangerous (can lose or corrupt data — confirm with the user first):**
- Dropping or renaming a column or table.
- Changing a column's type.
- Adding a NOT NULL column without a default to a populated table.
- Deleting rows, or any bulk `UPDATE`/`DELETE`.

**For anything dangerous:**
1. Confirm with the user before running it.
2. Make sure a backup or point-in-time recovery exists.
3. Prefer the expand-contract pattern instead of an in-place destructive change:
   add the new shape → backfill data → switch the app to use it → remove the old
   shape in a later, separate step once you're sure.
4. Keep migrations in version control (`supabase/migrations/`) so they're
   reviewable and repeatable, not run ad hoc in the dashboard.

Never run a destructive statement directly against production "just to see." If
you're unsure whether a change is destructive, treat it as if it is.

---

## Never fix by weakening security

The most damaging maintenance mistake is removing a safety control because it's
"in the way." When a security control blocks something, it is usually surfacing
the real problem, not causing it.

Do **not**, to make an error go away:
- Disable RLS or loosen a policy to "everyone" so a query returns rows.
- Skip or bypass Stripe webhook signature verification.
- Move a server-only secret to the client / behind `NEXT_PUBLIC_`.
- Replace a real authorization check with "is logged in".
- Add a service-role client to a browser code path.

If you genuinely believe a control must change, that is a decision for the user
to make with full context. Explain what the control does, why it's blocking
this, and the exact risk of changing it — then let them decide. Never do it
silently inside a "fix".

New tables added during maintenance get the same treatment as in a new build:
RLS on, owner-scoped policies, tested with a second user.

---

## Blast radius

Change the minimum. Before committing, especially when a fix touches shared
code:

- Find every caller of a function/component you modified and confirm none of
  them relied on the old behavior.
- Prefer a local fix over a change to shared code when both solve the problem.
- If a change has to be broad, say so explicitly and list what it touches so the
  user can review it deliberately.
- A bug fix is not a refactor. Log tempting cleanups under "later" rather than
  bundling them into the fix, so the diff stays reviewable and easy to roll back.

---

## Dependency changes

- Don't blind-upgrade a package hoping it fixes a bug. Read the changelog for
  breaking changes first.
- Change one dependency at a time and confirm the app still builds and runs.
- Pin versions rather than floating to "latest".
- Never add a package without confirming it actually exists on npm — hallucinated
  and typosquatted package names are a real supply-chain risk.
- After any dependency change, run the build and the core flow before committing.

---

## Rollback readiness

Every change should be easy to undo:
- Small, single-purpose commits with clear messages.
- Migrations that have a known reversal (or a documented recovery path for
  destructive ones).
- If deploying, know how to revert the deploy (Vercel keeps previous
  deployments — a rollback is one action).
