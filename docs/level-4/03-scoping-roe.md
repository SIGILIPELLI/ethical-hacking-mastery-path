---
description: "Scoping, contracts and rules of engagement: defining targets, authorization letters, testing windows and the pre-engagement work that protects everyone."
---

# Scoping, Contracts & Rules of Engagement

Level 1 lesson 1 said authorization and scope are what make the work legal. This lesson is the
practical craft of getting them right: the pre-engagement conversation and paperwork that define what
you'll test, how, when, and what happens when things go wrong. Done well, this phase protects the
client, protects you, and makes the whole engagement run smoothly. Done badly, it leads to legal
exposure, outages, or a test that measures the wrong thing. Experienced testers treat scoping as
seriously as exploitation — because a scoping mistake can end a career.

## Why pre-engagement is the most important phase

Everything downstream depends on it. A clear scope tells you what to test; a clear RoE tells you how
hard; a signed authorization makes it legal; an emergency contact saves the day when a test causes a
problem. Rushing this phase to "get to the fun part" is the single most common professional mistake,
and it is the one with legal consequences.

## Defining scope

Scope must be **explicit and bounded**. Work through, with the client:

- **Targets** — exact IP ranges/CIDRs, domains, subdomains, applications, APIs. List them. "The
  website" is not a scope; `https://app.example.com` and the `/api/v2` endpoints is.
- **Explicit exclusions** — systems that are off-limits even if reachable, and especially
  **third-party assets** the client uses but doesn't own (their payment processor, their SaaS, their
  cloud provider's control plane). The client **cannot authorize testing of systems they don't own**
  — you'd need the third party's permission too.
- **Test type** — external (internet-facing) vs internal (from inside the network) vs web-app vs
  wireless vs social engineering vs physical. Each needs its own authorization; social engineering
  and physical especially.
- **Knowledge level** — **black box** (no prior info), **grey box** (some credentials/info), **white
  box** (full access/source). This changes depth and realism and must be agreed.
- **Approach constraints** — production vs staging; are destructive tests (DoS, data modification)
  allowed (usually **not**)? Testing hours (to avoid business disruption)?

## Rules of Engagement (RoE)

The RoE governs *how*:

- **Testing window** — dates and hours testing is permitted. Outside it, you're unauthorized.
- **Permitted / forbidden techniques** — commonly: no DoS, no destructive actions, no social
  engineering unless separately scoped, no testing of production data integrity.
- **Rate/intensity limits** — to avoid knocking over fragile systems.
- **Handling of discovered data** — storage, redaction, destruction (ties to Level 4 lesson 1).
- **What to do on a critical finding** — stop-and-notify? A finding so severe (active breach, imminent
  risk) that you pause and call immediately.
- **Evidence of prior compromise** — the procedure if you find signs someone *else* is already in.
- **De-confliction** — how the client's blue team knows it's you, not a real attacker (or whether
  they're deliberately *not* told, for a red team).

## The legal/contract artefacts

- **Contract / Statement of Work (SOW)** — the commercial agreement, deliverables, timeline.
- **Authorization letter ("get out of jail free" letter)** — the document proving you're permitted,
  signed by someone with the **authority to grant it** (Level 1 lesson 1), naming the specific
  testers, targets, and dates. Carry it during the engagement.
- **NDA** — you'll see sensitive data; confidentiality is mutual and usually required.
- **Liability and insurance** — professional liability cover; agreed limits on responsibility if a
  test causes an outage despite care.

## Emergency contacts

Name a client contact reachable **24/7** for the duration. Testing can cause unexpected problems — a
fragile service falls over, a scan triggers an alert, you find an active breach. You need to reach
someone *immediately*. Agree in advance who, how, and for what.

## A worked example (reasoning)

A client says "test our web application." Good scoping turns that into: in-scope =
`app.example.com` and `api.example.com`; out-of-scope = the marketing site on a third-party CMS host,
the payment processor (a third party — not theirs to authorize), and the corporate email; type =
grey-box web-app test against **staging** (a production-equivalent environment) to avoid customer
impact; window = next two weeks, business hours excluded for load tests; no DoS, no social
engineering (not scoped); data = store encrypted, redact PII, destroy after; critical finding =
stop and call the named 24/7 contact; authorization letter signed by the CISO naming you and the
targets. Now you can test with confidence that every action is permitted and bounded.

## How It Actually Works

Why does the pre-engagement paperwork protect *everyone*, and why is a sloppy scope a genuine career
risk rather than mere bureaucracy? Recall the core legal fact (Level 1 lesson 1): computer-crime law
criminalises *unauthorized* access, and the target cannot tell a tester's packets from an attacker's
— so the only thing standing between "penetration test" and "felony" is a provable, bounded grant of
permission. Scope and the authorization letter *are* that grant, written down. If the scope is vague
("the website"), then the boundary of what you're permitted to touch is vague, and the first time you
test something the client didn't actually mean to include — a shared-hosting neighbour, a third-party
service, a system that turned out to be owned by a partner — you have committed unauthorized access
with no document saying otherwise. The precision of the scope is literally the precision of your
legal protection. This is also why third-party exclusions are non-negotiable: permission can only be
granted by the owner, so a client authorizing you to test *their* payment processor is like your
neighbour authorizing you to search *their* landlord's house — the signature is worthless because the
signer lacks the authority, and you'd be attacking a party who never consented.

The RoE and emergency contact protect everyone in a different way: they manage the fact that testing
is *inherently risky to live systems*. Even careful testing can crash a fragile service or trip an
incident response. The RoE pre-negotiates the risk the client is willing to accept (no DoS, staging
not production, rate limits), so that if something breaks *within* those agreed limits, it's a known
and accepted risk rather than a surprise the tester is blamed for — that's what the liability terms
and the "despite care" framing encode. The 24/7 contact exists because the worst moments
(you knocked over a server, or you found a real attacker already inside) are time-critical and can't
wait for the final report; a named, reachable human turns a potential catastrophe into a managed
event. Put together, pre-engagement is the phase where the irreducible dangers of the work — it's
legally identical to a crime, and it can break things — are converted into a bounded, consented,
and recoverable professional activity. That conversion is the entire reason the phase exists, and
why skipping it doesn't just risk a bad test; it risks the tester personally.

## Common mistakes and pitfalls

- **Vague scope.** "The website" or "our systems" leaves the legal boundary undefined. Enumerate
  exact targets and exclusions.
- **Authorizing third-party assets.** The client can't grant permission for systems they don't own.
  Exclude them explicitly; get separate permission if truly needed.
- **No authorization letter, or one signed by someone without authority.** The signer must be able
  to grant the permission. Carry the letter.
- **Skipping the RoE's "critical finding" and "prior breach" procedures.** When they're needed,
  they're needed *now* — define them up front.
- **No 24/7 emergency contact.** When a test causes a problem, you must reach someone immediately.
- **Testing outside the window.** Authorization is time-bounded; before/after the window you're
  unauthorized even against the same targets.
- **Treating scoping as a formality to rush.** It's the phase with the legal and operational
  consequences. Slow down here.

## Exercise

1. Take a vague request — "test our company's security" — and turn it into a precise scope: list the
   categories of information you'd pin down (targets, exclusions, type, knowledge level,
   constraints).
2. Draft a Rules of Engagement document for a web-app test: window, permitted/forbidden techniques,
   rate limits, data handling, critical-finding and prior-breach procedures, and the emergency
   contact.
3. Write the checklist of legal/contract artefacts you'd ensure exist before day one, and who must
   sign the authorization letter.
4. Explain why a client cannot authorize you to test their third-party payment processor, using the
   "authority to grant" idea.
5. In your own words, explain how precise scope *is* your legal protection, and why the RoE converts
   testing's inherent risk into an accepted, bounded risk.
