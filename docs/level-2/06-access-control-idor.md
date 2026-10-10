---
description: "Broken access control and IDOR: the most common serious web flaw — acting outside your permissions by changing an identifier or reaching a function."
---

# Broken Access Control & IDOR

Authentication proves *who you are*. **Authorization** (access control) decides *what you are
allowed to do*. Broken access control — letting a user act outside their intended permissions — is
consistently the most common serious web vulnerability, and it is often the easiest to find and the
most impactful: change a number in a request and read someone else's data. This lesson covers its
main forms, how to test them methodically, and why they are so pervasive. Practise on apps you run.

## The forms of broken access control

- **IDOR (Insecure Direct Object Reference)** — the app exposes a reference to an object (a record
  ID, filename, account number) and uses it without checking you're allowed that object. Change
  `?id=42` to `?id=43` and you get someone else's record.
- **Missing function-level access control** — an endpoint or admin function isn't protected on the
  server; a normal user who knows (or guesses) the URL can call it. The admin button is hidden in
  the UI, but the endpoint answers anyone.
- **Vertical privilege escalation** — a low-privileged user performs high-privileged actions (a
  regular user hitting `/admin/deleteUser`).
- **Horizontal privilege escalation** — a user accesses *another user's* data at the same privilege
  level (reading user 43's invoices as user 42). IDOR is the usual mechanism.
- **Forced browsing / parameter tampering** — reaching resources by guessing paths or flipping
  flags (`role=user` → `role=admin`, `isAdmin=false` → `true`).

## How to test it

Access control testing is about **identity vs. object**: for every object the app lets you
reference, ask "what if I reference one that isn't mine?" A disciplined approach:

1. **Map roles and objects.** Create (in the lab) at least two users — ideally two normal users and
   one admin. Note each one's objects (their orders, messages, profile) and their IDs.
2. **Catalogue every reference.** As user A, browse the app through your proxy and record every
   request that carries an identifier: `?id=`, `/users/42`, `account=…`, filenames, hidden fields,
   JWT claims.
3. **Swap identities, keep the object.** As user A (A's session), request user B's object ID. Do you
   get B's data? That's horizontal IDOR.
4. **Swap the function.** As a normal user, call an admin-only endpoint directly (from history, or
   by guessing). Does it work without your being admin? That's missing function-level control.
5. **Tamper with authorization data.** Flip a `role`/`isAdmin` parameter, or a value inside a token,
   and see if the server honours it.

```bash
# As USER A's session, request USER B's invoice (lab app). Do you get B's data?
curl -s -b 'session=USER_A_SESSION' http://app.lab/api/invoices/1002
# (1002 belongs to user B; 1001 is user A's)
```

If A's session returns B's invoice, you have a confirmed IDOR. Prove it with *one* adjacent record
you're not entitled to; you do not need to enumerate the whole database to establish the finding.

## A worked example

A banking demo app shows your statement at `/statement?account=1001`. In your proxy:

1. As user A (owns account 1001), the page loads A's statement — expected.
2. Change `account=1001` to `account=1002` and resend.
3. If A's session now shows account 1002's statement, the server authenticated A but never checked
   that A *owns* 1002. Horizontal privilege escalation via IDOR.
4. Impact: any customer can read any other customer's financial data by incrementing a number.
   Severity is high — network-reachable, trivial, confidentiality breach of sensitive data.
5. Fix: on every request, the server must verify the *current user* is authorised for the *requested
   object* — e.g. `WHERE account_id = :requested AND owner_id = :current_user`, or an explicit
   ownership/role check — and deny otherwise.

## Why predictable IDs make it worse (but aren't the bug)

Sequential IDs (`1001`, `1002`) make IDOR trivial to exploit, so people reach for UUIDs or random
tokens. That *raises the effort* (you can't just increment) but **does not fix the vulnerability** —
if the server still doesn't check ownership, anyone who *obtains* another object's ID (from a
referrer, a shared link, a log, another endpoint) can still access it. Unpredictable IDs are
"security through obscurity" here. The real fix is always the server-side authorization check.

## The defences

- **Deny by default.** Every request to a protected resource is denied unless an explicit check
  allows it. Don't rely on the UI hiding things.
- **Check authorization on the server, for every request, close to the data.** "Is *this user*
  allowed *this object/action*?" — not just "is this user logged in?"
- **Centralise the logic.** Scatter it across controllers and you'll miss endpoints. A shared
  authorization layer or middleware that every handler goes through is far more reliable.
- **Never trust client-supplied role/permission data.** Derive the user's role from the server-side
  session, never from a request parameter or an unverified token claim.
- **Log access-control failures** and alert on spikes (a sign of enumeration).
- Prefer unpredictable IDs as *defence in depth*, never as the control itself.

## How It Actually Works

Why is broken access control simultaneously the *most common* serious flaw and conceptually one of
the *simplest*? Because, unlike injection or XSS, there is **no single technical pattern that fixes
it** — authorization is inherently *per-resource, per-action, per-user application logic*, and it
must be re-stated correctly at every single place the app touches a protected object. Parameterised
queries kill all SQL injection with one technique; there is no equivalent one-liner for
authorization, because the question "should this user be allowed this?" has a different answer for
every endpoint and object. That means the developer has to remember to write the check *everywhere*,
and the vulnerability is simply the places they forgot. Humans forget consistently, so the flaw
appears consistently.

The deeper reason it's so often missing: authentication and authorization *feel* like one step but
are two. Logging a user in establishes identity and feels like "security is handled." But identity
only answers *who*; the server still has to answer *what this who may touch* on every request. An
app that checks "are you logged in?" and then serves whatever object ID you ask for has done
authentication and skipped authorization entirely — and because the happy-path UI only ever sends
*your own* IDs, everything looks correct in testing. The bug is invisible until someone sends an ID
the UI would never generate, which is exactly what a tester does. That asymmetry — works perfectly
for honest clients, fails for a crafted request — is why these bugs ship, and why "change the number
and resend" is such a productive test. The fix has to live on the server because, per the trust
boundary, the identifier in the request is attacker-controlled; only the server knows the true
mapping of users to objects, so only the server can decide.

## Common mistakes and pitfalls

- **Testing only with one user.** You can't find horizontal IDOR without a second account's objects
  to try to reach. Create multiple users in the lab.
- **Assuming a hidden/un-linked endpoint is protected.** "Not in the menu" is not access control.
  Call it directly.
- **Trusting UUIDs to fix IDOR.** They raise effort, not protection. The ownership check is the fix.
- **Checking authorization in the UI/JavaScript.** Client-side checks are bypassable; the server
  must enforce.
- **Over-collecting to prove it.** One adjacent record you shouldn't see proves the flaw; don't
  hoover up everyone's data.
- **Trusting role data from the client.** A `role=admin` parameter or an unverified token claim the
  server honours is itself the vulnerability.

## Exercise

1. In a practice app, create two normal users and (if possible) one admin. Catalogue each user's
   objects and their IDs.
2. As user A, attempt to read one of user B's objects by changing an identifier in a request.
   Record whether it succeeded — that's your horizontal IDOR test.
3. Find an admin-only function and call its endpoint directly as a normal user. Record whether
   function-level access control is enforced server-side.
4. Try tampering with a role/permission value (a parameter or token claim) and see if the server
   honours it. Explain the result.
5. In your own words, explain why there is no single "fix all access control" technique the way
   parameterised queries fix SQL injection, and why the flaw is invisible on the UI's happy path.
