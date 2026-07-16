# Security checklist

This is the reason the skill exists. Vibe-coded SaaS apps ship with a
predictable set of holes — missing row-level security, leaked keys, unverified
webhooks. Run the matching section during Phase 4 whenever you touch that area,
and run the whole thing in Phase 5 before calling anything shippable.

Report results plainly. "RLS not yet tested with a second user" is a useful
sentence. Silent assumptions are how the holes ship.

---

## 1. Supabase Row-Level Security (the big one)

Missing or broken RLS is the most common serious vibe-coding vulnerability —
whole classes of leaked-database incidents trace back to a table that had RLS
off or a policy that let any authenticated user read everyone's rows.

For every table holding user data:

- [ ] **RLS is ENABLED** on the table (it is OFF by default when you create a
      table via SQL — enabling it is a deliberate step).
- [ ] There is an explicit policy for each operation the app needs (select /
      insert / update / delete). No policy means no access; a too-broad policy
      means everyone's access.
- [ ] Policies are **owner-scoped**, typically `auth.uid() = user_id`, not just
      "is authenticated". "Logged in" is not the same as "allowed to see this
      row".
- [ ] **Tested with a real second user**: create user B, confirm B cannot read,
      update, or delete user A's rows. This is the only test that actually
      proves RLS works. Do it — don't assume.
- [ ] Any query that legitimately needs to bypass RLS uses the service-role
      client on the **server only**, for a specific reason, not as a shortcut to
      avoid writing policies.

## 2. Secrets and keys

- [ ] `.env.local` (and all `.env*` except `.env.example`) is git-ignored, and
      `git status` confirms no env file is tracked. Check this BEFORE any key is
      pasted, not after.
- [ ] No secret is committed anywhere in history. If one was, treat it as
      compromised: rotate the key, don't just delete the line.
- [ ] Service-role key, Stripe secret key, and webhook secret exist only in
      server code and host env vars — never in a client component, never behind
      `NEXT_PUBLIC_`.
- [ ] No secret is printed in full to logs or back to the user.
- [ ] Production secrets are set in the host (Vercel/Supabase) env config, not
      only on the local machine.

## 3. Auth and authorization

- [ ] Every protected route/page actually checks the session server-side.
      Hiding a link in the UI is not access control.
- [ ] Server actions and API route handlers re-check who the user is and whether
      they may do the thing — never trust a client-supplied user id or role.
- [ ] Authorization (can THIS user do THIS action on THIS row) is enforced, not
      just authentication (is someone logged in).
- [ ] No admin/privileged action is reachable by editing a request or guessing a
      URL.

## 4. Stripe / payments (if money is in v1)

- [ ] Webhook handler **verifies the Stripe signature** with the webhook secret
      before trusting any event. An unverified webhook endpoint is spoofable —
      anyone can POST a fake "payment succeeded".
- [ ] Entitlement (what the user gets after paying) is granted server-side in
      response to the verified webhook, not from a client-side "success" redirect
      that can be hit directly.
- [ ] Prices/amounts come from the server or Stripe, never from a client-sent
      value the user could tamper with.
- [ ] Test mode used throughout development; live keys only at real launch.

## 5. Dependencies and input

- [ ] Every package added is real and intentional — confirmed on npm, not
      hallucinated. Roughly a fifth of AI-suggested packages don't exist, and
      squatters register those names, so a made-up import can become a real
      malicious install.
- [ ] User-supplied input that hits the database goes through the Supabase
      client's parameterized queries (no string-concatenated SQL).
- [ ] User-supplied content rendered back is handled by React's default
      escaping; no unguarded `dangerouslySetInnerHTML` on user input.
- [ ] Server-side validation exists on anything important — client-side
      validation is UX, not security.

## 6. Before declaring shippable

- [ ] All sections above pass, RLS actually tested with two users.
- [ ] `npm run build` succeeds; no secret ends up in the client bundle.
- [ ] Rate-limiting or basic abuse protection considered for any expensive or
      public endpoint (e.g. anything that calls a paid API).
- [ ] A plain-language note to the user of what is covered and what is
      explicitly not (e.g. "no email verification yet", "no rate limiting on the
      generate endpoint").

If any box can't be checked, say so rather than working around it. The point of
this skill is that these specific things do not silently ship broken.
