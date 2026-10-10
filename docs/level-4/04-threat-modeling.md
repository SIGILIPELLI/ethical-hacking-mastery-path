---
description: "Threat modeling and attack-surface mapping: STRIDE, data-flow diagrams, trust boundaries and prioritising where to test so effort goes where the risk is."
---

# Threat Modeling & Attack Surface Mapping

A time-boxed test can't check everything, so where you spend effort matters. **Threat modeling** is
the structured way to decide: understand what you're protecting, who might attack it, how, and where
the weak points likely are — *before* and *during* testing. It turns "poke at random things" into
"test the paths that actually carry risk." It's a skill shared with defenders and architects (they
model to build securely; you model to test efficiently), and it makes you a far more effective tester
than tool-driven coverage alone.

## What threat modeling answers

Four classic questions (Adam Shostack's framing):

1. **What are we building / working on?** Understand the system — components, data, how they
   interact.
2. **What can go wrong?** Enumerate threats against each part.
3. **What are we going to do about it?** (Defender's question — for you, "where should I test?")
4. **Did we do a good job?** Validate the model and coverage.

As a tester you use it to **prioritise**: the highest-value targets are where sensitive assets meet
attacker-reachable surface across a trust boundary.

## Mapping the attack surface

The **attack surface** is every point where an attacker can interact with the system. Map it:

- **Entry points** — every input crossing a trust boundary: web forms, APIs, file uploads, network
  services, authentication flows (Level 2's trust-boundary idea, system-wide).
- **Assets** — what's worth protecting: customer data, credentials, payment info, intellectual
  property, availability of a critical service.
- **Trust boundaries** — where data/control passes between differently-trusted zones: internet↔DMZ,
  DMZ↔internal, user↔admin, service↔service.
- **Data flows** — how data moves between components (a **data-flow diagram**, DFD, makes trust
  boundaries visible).

The larger the surface, the more to test — and reducing it (disabling unused features, closing ports,
removing endpoints) is itself a defensive recommendation you can make.

## STRIDE — a threat taxonomy

**STRIDE** is a checklist for "what can go wrong" per component, each category the violation of a
security property:

| Threat | Violates | Example |
|--------|----------|---------|
| **S**poofing | Authentication | Pretending to be another user/system |
| **T**ampering | Integrity | Modifying data in transit or at rest |
| **R**epudiation | Non-repudiation | Denying an action with no audit trail |
| **I**nformation disclosure | Confidentiality | Leaking data to the unauthorized |
| **D**enial of service | Availability | Making a resource unavailable |
| **E**levation of privilege | Authorization | Gaining capabilities you shouldn't have |

Walk each component/data-flow through STRIDE and you systematically surface threat types you'd
otherwise miss. For a login flow: spoofing (credential stuffing), tampering (session manipulation),
information disclosure (username enumeration), elevation (privilege escalation after login) — each a
concrete test to run.

## Prioritising test effort

Risk ≈ **likelihood × impact**. Direct your time to:

- **Internet-reachable + sensitive** — the unauthenticated endpoint that touches customer data
  outranks an internal admin tool behind two firewalls.
- **Trust-boundary crossings** — where differently-trusted zones meet is where authorization and
  validation bugs cluster.
- **High-value assets** — follow the data: where PII, credentials and money live, test hardest.
- **Complex / custom code** — bespoke auth and business logic break more than well-trodden library
  paths.

This is also how you make a time-boxed test *honest*: you tell the client what you prioritised and
why, so "we focused on the internet-facing app handling payments" is a defensible allocation of a
limited budget.

## A worked example (reasoning)

Modeling a simple shop before testing: draw the DFD — browser → web app → database, with a payment
processor off to the side. Trust boundaries: internet↔web app, web app↔database, user↔admin, web
app↔payment (third party, out of scope). Assets: customer PII and order data in the DB; session
credentials; payment is the processor's problem. STRIDE across the login and checkout flows surfaces:
spoofing (auth), tampering (price/qty manipulation — a business-logic test), info disclosure (IDOR on
orders), elevation (customer→admin). You prioritise the internet-facing checkout and auth flows over
the internal admin panel, and you note the payment processor is out of scope. Your testing plan now
targets the highest-risk paths, and you can justify the allocation to the client.

## How It Actually Works

Why does a structured model beat an experienced tester's intuition about where to look — isn't
"test everything" the thorough approach? Because "everything" is never achievable in a real
engagement's time budget, so the real choice is always *what to skip*, and intuition skips badly. Left
to instinct, testers over-test the areas they find interesting or familiar and under-test the boring
but risky ones; they chase a clever exploit on a low-value system while an unauthenticated endpoint to
the customer database goes unchecked. Threat modeling forces the allocation to follow *risk* rather
than *interest* by making the two inputs to risk explicit and visible: the attack surface (where can
an attacker reach?) and the assets (what's worth protecting?). Risk concentrates precisely where those
two overlap — reachable *and* valuable — and a data-flow diagram with trust boundaries drawn on it
makes that overlap literally visible on the page. You test the crossings where high-value data meets
attacker-reachable input first, because that's where a bug does the most damage and where bugs
cluster (trust-boundary violations are the whole of Level 2).

STRIDE adds *completeness* to that focus. A human enumerating "what could go wrong" will recall the
threat types they've seen recently and forget the rest — maybe they think of injection and XSS but
not repudiation (no audit trail) or DoS. A taxonomy is a memory prosthesis: by walking each component
through all six categories, you surface threat classes your recent experience would have skipped, so
coverage doesn't depend on what happened to be top of mind. The combination — attack-surface mapping
for *focus* and STRIDE for *completeness* — is what lets a time-boxed test be both efficient and
defensible. And because the same model the defender builds to decide *what to protect* is the one you
build to decide *what to test*, your findings land in the client's own mental map of their system,
which makes them easier to act on. Modeling isn't overhead that delays the hacking; it's what makes
the limited hacking you can afford hit the targets that matter.

## Common mistakes and pitfalls

- **Skipping the model and testing by instinct.** You'll over-test the familiar and miss the risky-
  but-boring. Model first, even briefly.
- **Mapping surface without mapping assets.** Reachability alone doesn't rank risk; it's reachability
  *to something valuable*. Follow the data.
- **Ignoring trust boundaries.** The crossings are where authorization and validation bugs cluster;
  they deserve disproportionate attention.
- **Treating STRIDE as paperwork.** It's a prompt to find concrete tests; each category should
  produce specific things to try.
- **Modeling once and never updating.** New information from testing (an endpoint you didn't know
  about, a hidden admin function) feeds back into the model. Keep it live.
- **Not communicating prioritisation.** Tell the client what you focused on and why; it makes a
  time-boxed test honest and defensible.

## Exercise

1. Draw a data-flow diagram for a system you know (your Level 2 or 3 project target). Mark every
   component, data flow, and trust boundary.
2. List the assets worth protecting and, for each, where in the diagram it lives.
3. Walk two components (e.g. the login flow and one data-handling endpoint) through all six STRIDE
   categories, writing one concrete test per applicable category.
4. Rank your entry points by risk (likelihood × impact) and write a one-paragraph testing plan that
   justifies where you'd spend a limited time budget.
5. Explain, in your own words, why a structured model allocates effort better than intuition, using
   the "reachable × valuable" overlap idea.
