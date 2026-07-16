# Deterministic gates (hooks)

Skills bias behavior; hooks enforce it. A hook is a shell command Claude Code
runs automatically at defined points — it cannot be forgotten, talked out of,
or skipped under context pressure. Use hooks for the two failures that are
catastrophic and mechanical to detect: secrets reaching git, and broken code
being committed.

Offer to set these up during Phase 3 scaffold. Hooks live in the project's
`.claude/settings.json`. **Verify the current hook config syntax against the
Claude Code docs before writing it** — the schema has evolved; the concepts
below are stable, the exact JSON may not be.

## Gate 1 — block secrets from ever being committed (highest value)

Concept: a PreToolUse hook on Bash commands that match `git commit`, running a
check script; a non-zero blocking exit stops the commit and shows Claude the
reason, so it self-corrects.

Check script (`.claude/hooks/check-secrets.sh`), portable and dumb on purpose:

```bash
#!/bin/bash
# Block commit if any staged file looks like an env file or contains a likely secret
staged=$(git diff --cached --name-only)
if echo "$staged" | grep -qE '(^|/)\.env(\.|$)' ; then
  echo "BLOCKED: an .env file is staged. Unstage it and check .gitignore." >&2
  exit 2
fi
if git diff --cached | grep -qE '(sk_live_|sk_test_|service_role|SUPABASE_SERVICE_ROLE_KEY=ey|STRIPE_SECRET)' ; then
  echo "BLOCKED: staged changes contain what looks like a secret key. Remove it; use env vars." >&2
  exit 2
fi
exit 0
```

This is deliberately crude — it catches the common catastrophic cases
(env files, live Stripe keys, service-role JWTs), not everything. Crude and
always-on beats clever and forgotten.

## Gate 2 — typecheck/lint after every edit (fast feedback)

Concept: a PostToolUse hook on file edits that runs the project's cheap checks
(`tsc --noEmit` on the touched area, or `npm run lint --silent`) and surfaces
failures immediately, so errors are fixed at the moment of introduction rather
than discovered at build time three tasks later. Keep it FAST (seconds) — a
slow hook that runs on every edit makes the whole loop miserable. If the
project's typecheck is slow, run it on commit instead of on edit.

## What NOT to hook

- Do not hook the RLS second-user test or Playwright flows on every edit —
  too slow, wrong layer. Those belong at the Phase 4 verify step and Phase 5
  ship gate where the skill already places them.
- Do not add hooks that auto-fix silently; hooks should block-and-explain so
  Claude corrects with understanding, not paper over.
- Two or three hooks total. A hook wall that fires constantly trains everyone
  to ignore it.
