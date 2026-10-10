---
description: "Testing authentication and sessions: credential handling, brute-force and rate limiting, session fixation and hijacking, and password-reset flaws."
---

# Broken Authentication & Sessions

Authentication is how an app knows *who* you are; session management is how it *keeps* knowing
across requests. Break either and an attacker becomes someone else — the highest-impact outcome in
most apps. This OWASP category (sometimes "Identification and Authentication Failures") covers weak
credentials, missing brute-force protection, badly managed sessions, and broken password resets.
Test against your own practice apps and accounts only.

## What to test

### Credential handling

- **Weak password policy** — does the app allow `123456`, `password`, the username as the password?
- **Username enumeration** — does "user not found" differ from "wrong password"? If so, an attacker
  can discover valid usernames. The same leak hides in registration ("email already taken"), password
  reset, and even response timing.
- **Credentials in transit** — is login over HTTPS? Are credentials ever placed in a URL (and thus
  in logs/history)?
- **Credential storage** (if you can observe it in the lab) — are passwords hashed with a slow,
  salted algorithm (bcrypt/argon2/scrypt), or stored plaintext/MD5? (Covered deeply in Level 3's
  password lesson.)

### Brute-force protection & rate limiting

Can you submit thousands of login attempts unthrottled? Test for:

- account lockout or exponential backoff after repeated failures;
- rate limiting per IP / per account;
- CAPTCHA or step-up after suspicious activity;
- protection against **credential stuffing** (reusing breached username/password pairs) and
  **password spraying** (one common password across many accounts — which evades per-account
  lockout).

A safe lab test: script a handful of deliberately-wrong logins for an account you own and observe
whether the app slows, locks, or keeps accepting attempts.

```bash
# Against YOUR OWN practice app only — observe the response/status as attempts repeat
for i in $(seq 1 8); do
  curl -s -o /dev/null -w "attempt $i -> %{http_code}\n" \
    -X POST http://app.lab/login -d 'user=alice&pass=wrong'
done
```

If every attempt returns the same quick `200/401` with no lockout, there is no brute-force
protection — a finding.

### Session management

Once logged in, the session ID is the key to the account. Test:

- **Randomness** — is the session ID long and unpredictable, or guessable/sequential?
- **Session fixation** — does the app issue a *new* session ID at login, or keep the pre-login one?
  If it keeps it, an attacker who plants a known session ID in a victim's browser is logged in as
  them once they authenticate.
- **Cookie flags** — `HttpOnly`, `Secure`, `SameSite` present?
- **Logout & expiry** — does logout actually invalidate the session server-side, or just delete the
  cookie? Do sessions expire? Is the same token valid forever?
- **Session hijacking** — if you capture a session ID (e.g. via XSS, or in the lab by reading it),
  can you paste it into another browser and *be* that user? (Try it between your own two browser
  profiles in the lab.)

### Password reset

A weak reset flow is a backdoor around the password entirely:

- Are reset tokens long, random and single-use, with a short expiry?
- Does the reset leak whether an account exists?
- Can you tamper with the reset request to target another account (e.g. change a user-ID or email
  parameter)?
- Does using a reset link invalidate existing sessions?

## A worked example: session fixation

1. Visit the login page *without* logging in. Note the session cookie value, e.g.
   `session=FIXED123`.
2. In a second profile (the "victim"), set that same cookie and log in as a valid user.
3. Back in the first profile, with `session=FIXED123`, refresh a logged-in page.
4. **If** you are now logged in as the victim, the app failed to rotate the session ID on
   authentication — session fixation confirmed. The fix: issue a brand-new session ID at the moment
   of login and discard the old one.

## The defences

- Enforce a strong password policy; check candidates against breach lists; never store plaintext.
- Return **generic** auth errors ("invalid username or password") and keep response timing uniform.
- Rate-limit and lock out after repeated failures; add step-up/CAPTCHA; defend against spraying by
  limiting per-password-across-accounts too.
- Offer and encourage **multi-factor authentication (MFA)**.
- Generate long, random session IDs; **rotate the session ID on login and privilege change**;
  set `HttpOnly`, `Secure`, `SameSite`; expire idle and absolute sessions; invalidate server-side on
  logout.
- Make reset tokens long, random, single-use, short-lived; don't leak account existence; invalidate
  sessions on password change.

## How It Actually Works

Why does rotating the session ID at login defeat session fixation, and why is "keeping the same ID"
such a natural-seeming mistake? Because of *when* the identity binds to the session. A session is
just a server-side record keyed by the ID in the cookie. Before login, that record is anonymous;
after login, the server marks it "authenticated as alice." Session fixation exploits the gap: if the
ID doesn't change at login, an attacker who fixed the victim's cookie to a *known* value before
login now holds a cookie whose server-side record just got upgraded to "authenticated as alice" —
the attacker's known ID and the victim's new privileges point at the *same* record. The attacker
never needed the victim's password; they needed the ID to stay constant across the identity change.

Rotating the ID breaks the link: at login the server creates a *fresh* record with a fresh ID, binds
alice to it, and sends that new ID to the victim's browser. The attacker's fixed ID still points at
the old anonymous record, which is now worthless. The general rule — *rotate the session identifier
on any change of privilege* — exists because the identifier is a bearer token: whoever holds it is
treated as the subject it's bound to, so the moment the subject's privileges change, any
previously-shared copy of the identifier must stop working. The same reasoning explains why logout
must invalidate server-side: deleting the cookie in the victim's browser does nothing to a copy the
attacker already captured; only destroying the server-side record makes the bearer token
worthless.

## Common mistakes and pitfalls

- **Testing brute-force against accounts you don't own, or against real services.** Lab/own accounts
  only — unauthorised brute-forcing is an attack.
- **Missing username enumeration in non-login flows.** Registration and password reset leak account
  existence just as readily.
- **Assuming logout works.** Many apps delete the cookie client-side but leave the session valid
  server-side; capture the token before logout and test it after.
- **Overlooking session fixation because login "works."** It works for the user *and* the attacker —
  that's the bug. Check whether the ID actually changed at login.
- **Treating MFA as a cure-all.** MFA helps enormously but can be undermined by weak reset flows,
  session issues, or phishable factors. Test the whole lifecycle.
- **Reporting weak policy without impact.** Tie it to a demonstrated outcome (enumeration + no rate
  limit = feasible spraying) so it's actionable.

## Exercise

1. On a practice app, test for username enumeration across login, registration *and* password reset.
   Record any differing responses (text, status, or timing).
2. Script 8–10 wrong logins (as in this lesson) against your own practice account. Does the app
   throttle, lock, or do nothing? Record the behaviour and classify the finding.
3. Reproduce the session-fixation test between two browser profiles. State whether the app rotated
   the session ID at login, and explain the result.
4. Inspect a session cookie from a practice app for `HttpOnly`, `Secure` and `SameSite`. For each
   missing flag, describe one attack it would have hindered.
5. Explain, in your own words, why a session identifier is a "bearer token" and why that forces
   both rotation-on-login and server-side invalidation-on-logout.
