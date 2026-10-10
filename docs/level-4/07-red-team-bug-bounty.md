---
description: "Red teaming, penetration testing and bug bounty compared: three jobs with different goals, scopes, rules and deliverables, and staying legal in each."
---

# Red Team, Pentest & Bug Bounty Compared

"Offensive security" covers several distinct jobs that beginners often conflate. A **penetration
test**, a **red team engagement**, and **bug bounty hunting** share techniques but differ
fundamentally in goal, scope, rules, duration and deliverable — and those differences change what
you're allowed to do and how you should behave. Knowing which one you're doing (and what its rules
are) is part of staying legal and professional. This lesson draws the distinctions.

## Penetration test

- **Goal:** find as many vulnerabilities as possible in a defined scope, within a time box, and
  report them for fixing. Breadth and coverage.
- **Scope:** explicitly defined and usually announced; the blue team often knows it's happening.
- **Duration:** days to a few weeks.
- **Deliverable:** a comprehensive report of findings with severities and remediation (Level 4
  lesson 2).
- **Mindset:** thorough and systematic — work the methodology, cover the surface, document
  everything.

This is what the whole course has mostly taught. The value is a prioritised list of what to fix.

## Red team engagement

- **Goal:** emulate a *specific real-world adversary* to test the organisation's **detection and
  response** — not to list every bug, but to see whether the defenders catch a realistic attack
  pursuing a concrete objective ("reach the crown-jewel database," "get domain admin," "exfiltrate
  this file").
- **Scope:** often broad and goal-oriented; **stealth matters** — the blue team usually does *not*
  know, so the exercise measures real detection.
- **Duration:** weeks to months; slow and deliberate to avoid detection.
- **Deliverable:** a narrative of the attack path, what was detected and what wasn't, and
  recommendations to improve detection/response. Includes persistence and evasion (which a pentest
  usually avoids).
- **Mindset:** adversary emulation — be quiet, be patient, chase the objective, test the humans and
  the SOC as much as the tech.

A red team answers "if a real attacker targeted us, would we notice and stop them?" — a different
question from "what's vulnerable?" Often a **purple team** follows: red and blue collaborate openly
to improve defences using what the red team learned.

## Bug bounty

- **Goal:** find and report individual vulnerabilities in exchange for recognition or payment, on
  your own initiative.
- **Scope:** defined by the organisation's **bug-bounty program policy** — which assets are in scope,
  which are out, what techniques are forbidden, and the rules for testing. **The policy is your
  authorization**, and it is narrow and specific.
- **Duration:** open-ended; you choose when and what to test (within scope).
- **Deliverable:** individual, high-quality vulnerability reports, usually one at a time.
- **Mindset:** independent, opportunistic, deeply focused on finding one impactful, in-scope,
  previously-unknown bug — and reporting it responsibly.

Bug bounty is how many people practise legally on *real* systems: platforms like HackerOne and
Bugcrowd host programs where companies grant permission, within the policy, to test and report.
**Staying strictly within the program's scope and rules is non-negotiable** — stepping outside the
policy turns authorized research into unauthorized access.

## Side by side

| | Pentest | Red team | Bug bounty |
|--|---------|----------|------------|
| Primary goal | Coverage of vulns | Test detection/response | Find individual bugs |
| Scope | Defined, often known | Broad, goal-oriented | Program policy (narrow, specific) |
| Blue team aware? | Usually yes | Usually no (stealth) | Varies / usually yes |
| Duration | Days–weeks | Weeks–months | Open-ended |
| Stealth/evasion | Minimal | Central | Within policy |
| Deliverable | Full report | Attack narrative + detection gaps | Per-bug reports |
| Your authorization | Contract + RoE | Contract + RoE | The program policy |

## A worked example (same company, three jobs)

A bank hires out three engagements. The **pentest** systematically tests the online-banking app and
reports twenty findings by severity for the dev team to fix. Six months later a **red team** is given
the objective "reach the core banking ledger" with the SOC *not* informed; over two months they
phish a foothold, move quietly, and reach the objective — the deliverable is the path plus "the SOC
detected step 3 but not steps 4–9, here's how to close those gaps." Separately, an independent
researcher on the bank's **bug-bounty program** finds an IDOR in the public API, confirms it's
in-scope per the policy, reports it through the platform, and earns a bounty. Same bank, three goals,
three sets of rules, three authorizations.

## How It Actually Works

Why do these three jobs, built from the *same* techniques, demand such different behaviour — why
can't a tester just "hack well" and have it apply everywhere? Because the *question each engagement
answers* is different, and the question dictates the method. A pentest answers "what is vulnerable?"
— so its virtue is *coverage*, and coverage means working systematically, being loud is fine, and
listing everything is the deliverable. A red team answers "would we detect and stop a real
adversary?" — and you cannot measure detection if the defenders know you're coming and are watching
for you, so *stealth* becomes essential and the deliverable is not a bug list but a story of what
tripped alarms and what didn't. Finding every vulnerability would actually be *counterproductive* for
a red team: noisily scanning the whole network to enumerate bugs would get you caught immediately and
ruin the measurement. So the same action (a broad scan) is good practice in one job and a failure in
another, purely because the goal changed. This is why "which job am I doing?" is the first question —
it determines whether stealth, coverage, persistence or single-bug depth is the right behaviour.

The authorization structures differ for the same reason, and getting them wrong is where legality
lives. A pentest and red team are authorized by a *negotiated contract and RoE* between you and the
client — bespoke permission for a bespoke engagement. A bug bounty is authorized by a *standing,
public policy* the organisation offers to all researchers: the policy *is* the contract, and it is
deliberately narrow (specific in-scope assets, forbidden techniques, no social engineering, no DoS)
because the organisation is granting permission to strangers at scale and must bound it tightly.
That's why the cardinal rule of bug bounty is "stay in scope": the moment you test an asset the
policy doesn't list, you have no authorization for it, and — per Level 1 lesson 1 — unauthorized
access is the crime regardless of your good intentions or the fact that you report it. Bug bounty
feels casual and open, which makes the scope discipline *more* important, not less, because there's
no contract conversation to catch a misunderstanding. All three jobs reduce to the same foundation
the course started with: your authorization defines exactly what you may touch, and the shape of that
authorization — contract, RoE, or program policy — is just the shape of the permission for that kind
of work.

## Common mistakes and pitfalls

- **Treating a red team like a pentest.** Noisy, exhaustive scanning defeats the purpose (measuring
  detection) and gets you caught. Match behaviour to goal.
- **Treating a pentest like a red team.** A client paying for coverage doesn't want you to spend the
  budget being stealthy and finding one path; be thorough.
- **Going outside bug-bounty scope.** The policy is your authorization and it's narrow. Out-of-scope
  testing is unauthorized access, even if you report it.
- **Ignoring forbidden techniques in a program policy.** Many forbid DoS, social engineering,
  automated scanning. Read and obey the policy fully.
- **Reporting low-quality or duplicate bug-bounty findings.** Depth, impact, and clear reproduction
  matter; so does checking it isn't already known.
- **Assuming one authorization covers another.** A pentest contract doesn't authorize bug-bounty-
  style testing of other assets, and vice versa. Each job, its own permission.

## Exercise

1. Build the comparison table from memory (goal, scope, blue-team awareness, duration, stealth,
   deliverable, authorization) for all three.
2. For a single target organisation, write a one-line objective appropriate to each of the three
   engagement types.
3. Find a real public bug-bounty program policy (HackerOne/Bugcrowd). List its in-scope assets, two
   out-of-scope items, and two forbidden techniques.
4. Explain why exhaustive scanning is good practice in a pentest but a mistake in a red team, using
   the "question each engagement answers" idea.
5. In your own words, explain why "stay in scope" is *more* critical in bug bounty than in a
   contracted pentest, referencing how each is authorized.
