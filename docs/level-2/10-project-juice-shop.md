---
description: "Level 2 capstone: a structured web app assessment of OWASP Juice Shop, working the Top 10 methodically into a written findings report."
---

# Project — Assessing OWASP Juice Shop

This project puts the whole level together into a real workflow: a structured web application
assessment of **OWASP Juice Shop**, the deliberately vulnerable app the OWASP project maintains for
exactly this purpose. You'll map the app, work the OWASP Top 10 methodically through an intercepting
proxy, prove each finding with a harmless proof-of-concept, and write it up as a findings report —
the Level 2 equivalent of Level 1's discovery report.

Juice Shop is safe and legal to attack because it exists to be attacked, runs entirely on your own
machine, and ships with a wide range of planted vulnerabilities across every Top 10 category.

## Setup

Run Juice Shop locally (any one of these; all local, all free):

```bash
# Option A — Node (if you have Node installed)
# (fetch the project per its README, then:)
npm start                      # serves on http://localhost:3000

# Option B — Docker
docker run --rm -p 3000:3000 bkimminich/juice-shop
```

Then browse `http://localhost:3000` through Burp or ZAP (Level 2 lesson 2), with scope set to
`localhost:3000`. You are attacking `localhost` — your own machine, nobody else's.

!!! note "Honest note on this course's environment"
    This course was written on a machine without Docker and without a running Juice Shop instance,
    so the specific flags you find are yours to discover — this project deliberately does **not**
    print "the answers" or invent challenge solutions. Juice Shop also has an official companion
    guide (the "Pwning OWASP Juice Shop" book) and a built-in score board; use them to check your
    work. What follows is the professional *method* to apply, which is the transferable skill.

## The assessment method

Work category by category. For each, (a) locate candidate inputs, (b) test safely, (c) prove impact
with a harmless PoC, (d) record evidence and the fix.

### 1. Reconnaissance & mapping

- Browse every feature through the proxy to populate the sitemap.
- Find the REST API endpoints the SPA calls; look for an OpenAPI spec.
- Note the roles (anonymous, customer, admin) and create two customer accounts for access-control
  tests.

### 2. Injection (lesson 3)

- Test login and search inputs for SQL injection with a single quote, then boolean logic.
- Prove with an error or a logic bypass — not by dumping the whole database.

### 3. XSS (lesson 4)

- Find reflected inputs (search, URL parameters) and a stored sink (a field persisted and shown to
  others). Prove execution with `alert(document.domain)`.

### 4. Broken authentication & sessions (lesson 5)

- Test for username enumeration, missing rate limiting, weak password policy, and the password-reset
  flow (the security-question mechanism is a deliberate weakness class).
- Inspect the JWT the app issues: decode it, check what's inside, test whether tampering is detected.

### 5. Broken access control / BOLA (lessons 6 & 9)

- With customer A's session/token, try to read customer B's basket, orders or profile by changing
  IDs.
- Try to reach admin-only functionality as a customer.

### 6. SSRF / upload / traversal (lesson 7)

- Test any file-upload and any URL-fetch feature for the flaws in lesson 7, safely.

### 7. Misconfiguration & components (lesson 8)

- Check security headers (`curl -sI`), look for exposed files and verbose errors, and fingerprint
  component versions.

### 8. API & business logic (lesson 9)

- Test the API directly for BOLA, mass assignment (add fields to a profile/basket update), and
  missing authorization.
- Look for business-logic flaws: negative quantities, price tampering, applying a coupon multiple
  times, ordering with an empty/negative total.

## Deliverable: the findings report

Produce a report with:

1. **Executive summary** — plain-language overview: how many findings, the worst ones, overall
   posture. (One paragraph a non-technical manager understands.)
2. **Scope & methodology** — target (`localhost:3000`), tools, dates, what you tested.
3. **Findings**, each with:
   - a clear title and the OWASP category / CWE;
   - **severity** with a CVSS vector and your reasoning;
   - **steps to reproduce** (the exact request(s) — this is why you worked through the proxy);
   - **evidence** (the request/response, a screenshot of the harmless PoC);
   - **impact** in business terms;
   - **remediation** — the specific fix (parameterise, encode, add the ownership check, set the
     header…).
4. **Findings table** — a one-line-per-finding summary sorted by severity.

Order findings by severity, most serious first. Aim for quality over quantity: five well-evidenced,
correctly-rated findings beat twenty vague ones.

## Self-check

- Every finding has reproducible steps someone else could follow.
- Every severity has a CVSS vector and a one-line justification.
- Every PoC is *harmless* (an `alert`, a bypass, one adjacent record) — you did not exfiltrate real
  data or damage the app.
- Every finding has a concrete, correct remediation.
- The executive summary is readable by a non-technical stakeholder.
- You stayed in scope (`localhost:3000`) throughout.

## How It Actually Works

Why structure an assessment by category and insist on reproducible, harmlessly-proven findings,
rather than just listing "bugs"? Because the report, not the hacking, is the product — and a report
is only valuable if it drives *fixes*. A finding a developer can reproduce in thirty seconds from
your steps gets fixed; a vague "the login is insecure" gets ignored or argued with. Working
category by category guarantees *coverage*: you can state you tested injection, XSS, access control
and the rest, so the client knows the test was systematic rather than whatever caught your eye. That
coverage claim is only credible because you recorded negative results alongside positives, exactly
as in the Level 1 project.

Proving impact harmlessly is both an ethics and a credibility matter. `alert(document.domain)`
proves XSS executes in the app's origin without stealing anyone's session; one adjacent record
proves IDOR without dumping a database; a logic bypass proves SQLi without exfiltration. The proof
establishes the vulnerability is real (not a scanner's guess) while respecting the rule from Level 1
lesson 1 — *prove impact, don't cause it*. And because you captured each proof through the proxy,
the "steps to reproduce" write themselves from your evidence. This is the whole arc of the course in
miniature: methodical discovery, confirmed findings, and a deliverable that makes the application
safer — which is the entire point of ethical hacking.

## Exercise

1. Stand up Juice Shop locally and proxy it with scope set to `localhost:3000`. Map the app and
   create two customer accounts.
2. Find and safely prove at least one finding in **each** of these categories: injection, XSS,
   broken access control, and misconfiguration. Capture the request/response evidence for each.
3. Write the full findings report (all four sections), ordering findings by severity with CVSS
   vectors.
4. Run your report against the self-check list and fix every gap.
5. Compare your findings against Juice Shop's built-in score board or the official companion guide.
   Note any category you *missed* and, in two sentences, why a methodical category-by-category pass
   would have caught it.
