---
description: "Writing the penetration test report: executive summary, findings with reproducible steps, evidence, risk ratings, and remediation that gets acted on."
---

# Writing the Penetration Test Report

The report is the product. The client paid for a document that makes their systems more secure, and
no matter how skilled the testing, an unclear report wastes it. A great report is read by two very
different audiences — executives who decide budgets and engineers who apply fixes — and it has to
serve both. This lesson is how to write one that gets findings *fixed*, which is the only measure of
success that matters.

## Who reads it, and what they need

| Reader | Wants | Section that serves them |
|--------|-------|--------------------------|
| Executives / management | Overall risk, business impact, "how bad, what now" | Executive summary |
| Security / engineering | Exactly what's wrong, how to reproduce, how to fix | Detailed findings |
| Compliance / auditors | Scope, methodology, evidence of a thorough test | Scope & methodology |
| Future testers (retest) | What was found, so they can verify fixes | Findings + reproduction |

Write each section for its audience. The executive summary must be readable by someone non-technical;
the findings must be precise enough for an engineer to reproduce and fix.

## Report structure

### 1. Executive summary

One page, plain language, no jargon. It states: what was tested and when, the overall security
posture, the number of findings by severity, the most serious issues in business terms, and the top
recommendations. A CFO should finish it understanding the risk and what to prioritise. Avoid
tool names and exploit details here — translate "unauthenticated RCE on the billing server" into
"an attacker on the internet could take full control of the system that processes payments."

### 2. Scope & methodology

What was in and out of scope, the testing window, the approach/standard followed (PTES, OWASP WSTG),
and the tools used. This establishes the test's boundaries and credibility, and matters for
compliance. Note what was *not* tested so no one assumes false coverage.

### 3. Findings (the core)

Each finding, ordered by severity, contains:

- **Title** — clear and specific ("Unauthenticated SQL injection in the login form").
- **Severity** — a rating (Critical/High/Medium/Low/Info) *with a CVSS vector* and a one-line
  justification, adjusted for the client's environment (Level 1 lesson 9).
- **Affected asset(s)** — the specific host/URL/endpoint.
- **Description** — what the vulnerability is, in terms the team can understand.
- **Steps to reproduce** — the exact requests/commands, so an engineer can see it themselves. This is
  what turns a claim into something undeniable and fixable.
- **Evidence** — screenshots, request/response, output (with PII redacted).
- **Impact** — what an attacker achieves, in business terms.
- **Remediation** — the specific, correct fix ("use parameterised queries"), plus references.

### 4. Findings summary table

One line per finding — ID, title, severity — for a scannable overview.

### 5. Appendices

Full tool output, the engagement log, detailed evidence, methodology references.

## Severity: rate for the client, not the textbook

Use CVSS as the backbone but adjust for *this* environment (the Environmental metrics): a Critical on
an isolated test box may be a Medium in reality; a Medium info-leak that enables the whole attack
chain may warrant higher attention. Always explain the reasoning. And where findings *chain*
(Level 3 project), say so — a report that rates five "mediums" without noting they combine into
domain compromise has under-served the client.

## Remediation that gets acted on

- Be **specific and correct**: "parameterise the query in `login()`" beats "sanitise inputs."
- Be **actionable**: name the fix, link authoritative guidance (OWASP cheat sheets, vendor docs).
- **Prioritise**: tell them what to fix first, and (from the chain analysis) which single fixes break
  the most attack paths most cheaply.
- Distinguish **quick wins** from **strategic fixes** (a config change today vs. an architecture
  change over a quarter).

## Tone and accuracy

- **Professional and blame-free.** The report documents system weaknesses, not people's failures.
- **Accurate above all.** Never overstate severity to look impressive, never claim an unvalidated
  finding, never fabricate evidence. Your credibility is the entire value of the report; one
  invented finding poisons all of it.
- **Honest about limitations.** Time-boxed tests don't find everything; say what wasn't covered. "No
  findings" means "none found in scope and time," not "provably secure."

## A worked example (a single finding, abbreviated)

```
[HIGH] SQL Injection in Login Form (CWE-89)   CVSS: AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N (8.1)
Asset:  https://app.lab/login  (username field)
Summary: The username field is concatenated into an SQL query, allowing authentication bypass
         and database access.
Reproduce:
  1. POST to /login with username:  ' OR 1=1 -- 
  2. Observe authentication succeeds without a valid password (evidence: screenshot A).
Impact: An unauthenticated attacker can bypass login and read/modify the application database,
        which contains customer PII.
Remediation: Use parameterised queries / prepared statements for all database access (see OWASP
        SQL Injection Prevention Cheat Sheet). Add least-privilege DB accounts as defence in depth.
```

Severity is justified, reproduction is exact, impact is in business terms, and the fix is specific
and referenced.

## How It Actually Works

Why does report *quality* determine the security value of an entire engagement — why isn't finding
the vulnerabilities the hard part? Because a vulnerability that isn't *fixed* provided no security
benefit, and whether it gets fixed is almost entirely a function of the report. Fixing a finding
requires two different people to act: a decision-maker has to prioritise and fund the work, and an
engineer has to implement the change. Each will act only if the report speaks to them. The executive
won't allocate resources to "unauthenticated RCE" they don't understand, but will to "an attacker
could take over the payment system" — same finding, different framing, opposite outcome. The engineer
won't fix "the login is insecure," but will fix a specific query shown with exact reproduction steps
and a named remediation they can implement in an hour. The report is the *interface* between the
tester's knowledge and the organisation's action, and like any interface, it fails if it's mismatched
to the party on the other side — hence two audiences, two registers, in one document.

Accuracy is load-bearing for a subtler reason: the report asks the organisation to spend real money
and effort on the tester's say-so, so it runs entirely on *credibility*. Every overstated severity
that an engineer investigates and finds exaggerated, every "finding" that turns out to be a
false positive the tester didn't validate, teaches the client to discount the report — and once
they're discounting it, even the real Criticals may not get fixed. This is why validation (Level 3
lesson 1), minimum-sufficient proof (lesson 1), and honest limitations matter so much: they're not
pedantry, they're what keeps the report *trusted*, and a trusted report is the only kind that
changes behaviour. The whole penetration test — the recon, the exploitation, the privilege escalation
— exists to produce evidence; the report is where that evidence either becomes action or evaporates.
That's why professionals spend a large fraction of the engagement on it.

## Common mistakes and pitfalls

- **A technical executive summary.** Executives don't read exploit chains; translate to business
  risk or they'll disengage from the whole report.
- **Findings without reproduction steps.** Unreproducible findings get argued with and ignored.
  Exact steps make them undeniable and fixable.
- **Vague remediation.** "Sanitise inputs" is not a fix. Name the specific change and link guidance.
- **Severity inflation (or deflation).** Rate honestly, adjust for environment, and explain. Credibility
  is everything.
- **Dumping raw tool output as "the report".** Scanner output is not a report; synthesise, validate,
  prioritise.
- **Ignoring chains.** Note where mediums combine into a critical path, and which fix breaks it.
- **Claiming security.** "No findings" ≠ "secure." State the time-box and coverage limits.

## Exercise

1. Take one finding from your Level 2 or Level 3 project and write it up in the full finding format
   (title, CVSS vector + justification, asset, description, reproduction, evidence, impact,
   remediation).
2. Write a one-page executive summary for that project in plain language — no tool names, no jargon.
   Have someone non-technical read it and tell you the risk back; revise until they get it right.
3. Build the findings summary table for the project, sorted by severity.
4. For your highest finding, write remediation that is specific, actionable, referenced, and
   prioritised.
5. Explain, in your own words, why report *accuracy* is load-bearing — how a single overstated or
   unvalidated finding can reduce the chance that your real findings get fixed.
