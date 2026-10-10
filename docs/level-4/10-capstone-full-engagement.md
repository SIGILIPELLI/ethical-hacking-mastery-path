---
description: "Capstone: run a complete ethical-hacking engagement end to end in your lab — scoping, recon, testing, exploitation, reporting and debrief."
---

# Capstone — A Complete Engagement End to End

This capstone ties the entire course together: you will run a **full engagement** against your lab,
from the pre-engagement paperwork through to a professional report and debrief — exactly as a real
penetration test flows. Everything you've learned has a place here: authorization and scoping
(Level 1 + Level 4), the lifecycle and recon (Level 1), web and network testing (Levels 2–3),
post-exploitation ethics (Level 4), and the report (Level 4). The deliverable is a complete,
professional engagement package you could show an employer as portfolio evidence (Level 4 lesson 8).

Run it entirely in your own lab. The point is to practise the *whole* process with discipline, not
just the exciting middle.

## Build the engagement target

Assemble a small but realistic lab "organisation":

- A **web application** (Juice Shop / DVWA, or a VulnHub web box) as the internet-facing surface.
- One or two **infrastructure hosts** (Metasploitable / a VulnHub box), ideally with a **pivot**
  path to an internal-only host.
- Optionally a small **AD lab** (DC + workstation) as the internal crown jewels.

Snapshot everything clean. This stands in for a client environment.

## Phase 1 — Pre-engagement (Level 4 lesson 3)

Write the engagement paperwork *as if for a client*, even though you own the lab:

- **Scope:** the exact in-scope targets (your lab CIDRs/hosts/app URLs) and explicit exclusions.
- **Rules of Engagement:** window, permitted/forbidden techniques, rate limits, data handling,
  critical-finding and prior-breach procedures, and a (notional) 24/7 emergency contact.
- **Objective:** what "maximum impact" means here (e.g. "reach the internal database / Domain
  Admin").
- **Authorization statement:** a one-paragraph mock authorization letter naming the tester
  (you), the targets, and the window.

This phase is graded on precision — vague scope is the professional failure of Level 4 lesson 3.

## Phase 2 — Threat model (Level 4 lesson 4)

Draw a data-flow diagram of your lab org, mark trust boundaries and assets, run key components
through STRIDE, and write a one-paragraph testing plan prioritising the highest-risk paths. This
directs where you'll spend effort.

## Phase 3 — Recon, scanning & enumeration (Level 1)

Discover hosts, scan all ports, version-detect, and enumerate every service into the per-host
attack-surface table. Map versions to candidate vulnerabilities (CVE/CVSS) and mark them
unvalidated.

## Phase 4 — Vulnerability analysis & validation (Level 3 lesson 1)

Validate the promising candidates; separate real from false-positive; prioritise by severity ×
reachability. Record reasoning.

## Phase 5 — Exploitation & post-exploitation (Levels 2–3, Level 4 lesson 1)

Work the web app (OWASP Top 10, Level 2) and the infrastructure (exploitation, privilege escalation,
pivoting, AD — Level 3). For each foothold, perform **ethical** post-exploitation: assess impact,
gather **minimum-sufficient proof**, harvest credentials for the next hop, and clean up. Maintain a
timestamped engagement log throughout. Snapshot before destructive steps.

## Phase 6 — Reporting (Level 4 lesson 2)

Write the full report:

- **Executive summary** (one page, plain language, business risk).
- **Scope & methodology.**
- **Attack narrative** — the kill chain from foothold to objective.
- **Findings** — each with title, CVSS vector + justification, asset, description, reproduction,
  evidence (PII redacted), impact, remediation.
- **Highest-value fixes** — which one or two changes break the chain earliest.
- **Findings summary table** (by severity).
- **Appendices** — the engagement log and supporting evidence.

## Phase 7 — Debrief

Write a short debrief as if presenting to the client: the three most important takeaways, the top
remediations in priority order, and what you'd test next time. This mirrors the real end of an
engagement and the purple-team collaboration a mature client wants.

## The deliverable package

A single directory containing: the scope/RoE/authorization docs, the threat model, the engagement
log, the full report, and the evidence. This *is* portfolio material — a complete, professional
engagement you ran end to end.

## Self-check (the whole course, applied)

- **Authorization & scope:** precise, written, obeyed throughout; nothing out-of-scope touched.
- **Methodology:** every lifecycle phase present; coverage demonstrable (including negative results).
- **Validation:** findings are confirmed, not scanner guesses; false positives noted.
- **Ethics:** minimum-sufficient proof; no real data exfiltrated; cleaned up and logged.
- **Report:** executive summary a non-technical reader understands; findings reproducible; severities
  justified; remediation specific; the chain and its cheapest break identified.
- **Honesty:** nothing fabricated; limitations and untested areas stated.

If all six hold, you have demonstrated the full skillset this course set out to teach.

## How It Actually Works

Why does a *complete* engagement — paperwork, model, recon, validation, ethics, report, debrief —
demonstrate competence in a way that "I rooted the box" never can? Because the box is the part that's
*fun*, and fun is not the job. The course's recurring "How It Actually Works" sections have all
pointed at one idea from different angles: the technical skill of finding a way in is necessary but
not sufficient, because the *value* of ethical hacking is produced almost entirely by the parts
around the exploit. Authorization and scope are what make the work legal rather than criminal.
Validation is what makes a finding trustworthy rather than a scanner's guess. Ethical post-
exploitation is what makes the test a measurement rather than a second attack. The report is what
converts your knowledge into the client's action — the only place security actually improves. The
debrief is what makes the improvement stick. A capstone that runs *all* of these proves you can do
the job; a capstone that only exploited a box proves you can do the one part that, done alone, is
worthless (or worse, illegal). This is precisely why the respected hands-on certifications require a
report, and why employers prize a documented engagement over a list of rooted machines.

Step back and the whole course has a single shape. It began by establishing that offensive technique
is identical to a criminal's and that *permission* is the only thing separating the two — so it built
you a legal lab to practise in. It then taught the technique itself (recon, web, infrastructure)
always alongside the discipline that keeps it ethical (in-scope, lab-only, minimum-sufficient proof).
And it ended with the professional craft (scoping, reporting, law, careers) that turns technique into
a trustworthy service. The capstone makes you walk the whole arc once, under your own discipline,
because that arc *is* ethical hacking: not the ability to break in, but the ability to break in *with
permission, without harm, and in a way that makes the target more secure.* Everything else is detail;
that sentence is the job. If your capstone package embodies it, you are no longer learning to be an
ethical hacker — you are practising as one.

## Exercise

1. Build the lab engagement target (web + infrastructure, ideally a pivot and/or AD) and snapshot it
   clean.
2. Run all seven phases end to end, maintaining the paperwork, threat model, timestamped log, and
   evidence as you go.
3. Produce the complete deliverable package (scope/RoE/authorization, threat model, log, report,
   evidence) in one directory.
4. Run the package against the six-point self-check and fix every gap until all hold.
5. Write the debrief, then write one honest paragraph on what you found hardest in the *whole
   engagement* (not just the exploitation) and how you'd improve it — the reflection a professional
   does after every job.
