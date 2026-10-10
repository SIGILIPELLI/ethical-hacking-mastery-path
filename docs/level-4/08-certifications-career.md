---
description: "Certifications and the ethical-hacking career path: what OSCP, PNPT, CEH and others test, building a lab portfolio, and landing the first role."
---

# Certifications & Career Path

Ethical hacking is a field you can genuinely break into through demonstrated skill, and certifications
are one of the main ways to prove that skill to employers — but not all certs are equal, and none
substitute for hands-on ability. This lesson maps the major certifications (what each actually tests),
how to build a portfolio that complements them, and the realistic path into a first role. It's
honest about what helps and what's marketing.

## The major certifications

| Cert | Focus | Style | Reputation |
|------|-------|-------|------------|
| **CompTIA Security+** | Broad security fundamentals | Multiple choice | Common entry/HR filter; not hands-on |
| **CEH** (Certified Ethical Hacker) | Broad hacking concepts | Mostly multiple choice (practical option exists) | Widely *recognised* by HR, often criticised as theory-heavy |
| **OSCP** (Offensive Security Certified Professional) | Hands-on network/web exploitation | 24-hour practical exam | Highly respected; the classic hands-on benchmark |
| **PNPT** (Practical Network Penetration Tester) | Realistic engagement incl. AD + report | Practical exam + a real report debrief | Respected, engagement-realistic, well-priced |
| **eJPT** | Entry-level practical | Practical | Good first practical cert |
| **OSWE / OSEP / OSED** | Advanced (web, evasion, exploit dev) | Hard practical | Advanced specialisations |
| **GPEN / GWAPT** (SANS/GIAC) | Pentest / web | Exam | Respected, expensive |
| **CRTO / CRTP** | Red team / AD | Practical | Strong for red-team/AD roles |
| Cloud: **cloud provider security certs** | Cloud security | Varies | Valuable as cloud testing grows |

Two things to internalise:

- **Hands-on certs (OSCP, PNPT, the practical OffSec/Zero-Point tracks) carry the most weight** with
  technical interviewers, because they prove you can actually do the work, not just recognise terms.
- **Multiple-choice certs (Security+, standard CEH) help pass HR filters** but won't convince a
  technical panel on their own. They have a place — getting the résumé through the first gate — but
  know what they do and don't prove.

The best-respected hands-on exams (notably OSCP) are famous for their difficulty and for the
**report** requirement — you must document your compromises professionally, which is the skill Level
4 lesson 2 taught. "Try harder" is the OSCP cliché precisely because it rewards persistence and real
problem-solving over memorisation.

## Skills matter more than certs

A certificate opens a door; demonstrated skill gets you through it. Build evidence:

- **A home lab** (Level 1 lesson 2) you actually use — vulnerable VMs, an AD lab, documented
  compromises.
- **Practice platforms** — **Hack The Box**, **TryHackMe**, **VulnHub**, **PortSwigger Web Security
  Academy** (free and excellent for web), and CTFs. These are legal, real-feeling practice, and your
  progress is portfolio-worthy.
- **Bug bounty** (Level 4 lesson 7) — legal practice on real systems, within program scope, with
  reportable results.
- **Write-ups and a blog** — documenting how you solved a box or found a bug demonstrates both skill
  *and* the communication/reporting ability employers desperately want.
- **Open-source contributions / tools** — even small ones show initiative and competence.

## The realistic career path

Few people start *as* a penetration tester. Common routes in:

1. **Adjacent IT/security roles first** — SOC analyst, sysadmin, network/support, help desk,
   developer. These build the systems knowledge that makes a good tester, and give you a security-
   relevant job while you skill up. (The defensive side is the
   [Cybersecurity Mastery Path](https://sigilipelli.github.io/cybersecurity-mastery-path/).)
2. **Junior pentester / security analyst** — once you have an OSCP/PNPT-level skillset and a
   portfolio.
3. **Specialise** — web, cloud, red team, mobile, exploit dev, hardware — as you find what you enjoy
   and where demand is.

The field rewards **continuous learning**: techniques, tools and defences change constantly. Follow
research, keep labbing, keep reading.

## A realistic first-year plan (one sensible path)

- Months 1–3: fundamentals + the PortSwigger Academy (free) + a home lab; aim for eJPT or steady
  TryHackMe/HTB progress.
- Months 3–9: work toward **OSCP or PNPT**; build and document a lab portfolio; start write-ups.
- Alongside: a security-adjacent job if you don't have one (SOC, IT, dev) for income and experience.
- Months 9–12: apply for junior roles with the cert, the portfolio, and the write-ups; practise
  explaining your methodology in interviews.

This is *a* path, not *the* path — people arrive from development, IT, the military, and self-study.
The constants are hands-on skill, documentation ability, and persistence.

## How It Actually Works

Why do hands-on certifications and a lab portfolio outweigh multiple-choice credentials with
technical employers, when all of them are "certifications"? Because the thing an employer is actually
trying to predict is *can this person do the job*, and the different exam formats are wildly different
*predictors* of that. A multiple-choice exam tests recognition — can you identify the right answer
when it's listed — which correlates weakly with the open-ended, no-answer-key reality of a real
engagement where nobody tells you the box is vulnerable to anything. A 24-hour practical exam like
OSCP tests the actual loop of the job: enumerate an unknown target, form hypotheses, fail, adapt,
eventually get in, and *write it up* — which is almost exactly what a pentest is. So the hands-on cert
is a better predictor not because it's "harder" for its own sake, but because its format *matches the
work*, and a technical interviewer knows this. The report requirement on the best exams is the clue:
they're testing the deliverable (Level 4 lesson 2), because a tester who compromises a box but can't
document it reproducibly hasn't produced the thing the client pays for.

This also explains why a portfolio can rival or exceed a certificate, and why the field is unusually
open to self-taught entrants. Offensive security has an objective, demonstrable output: a documented
compromise, a bug-bounty report, a CTF write-up. Unlike fields where competence is hard to show
without a credential, here you can simply *exhibit* the skill — "here are ten boxes I rooted with
write-ups, here's a bug I found and reported" — and that evidence speaks directly to the predictive
question an employer cares about. Certifications remain useful because they're a standardised,
third-party signal that scales (HR can filter on them, and a known hard exam is a trusted proxy), but
they're a proxy; the portfolio is the thing itself. The career path through adjacent roles works for
the same underlying reason: SOC, sysadmin and dev jobs build the deep systems understanding that
separates someone who runs tools from someone who understands *why* an attack works — which is the
entire difference this course has tried to teach, lesson by lesson, with every "How It Actually
Works."

## Common mistakes and pitfalls

- **Collecting certs without hands-on skill.** A wall of multiple-choice certs won't survive a
  technical interview. Pair any cert with demonstrable practice.
- **Assuming CEH or Security+ alone makes you a pentester.** They pass HR filters; they don't prove
  you can do the work. Know what each proves.
- **Skipping the report skill.** The best exams require it and so does the job. Practise documenting
  every box you do.
- **Not building a portfolio.** Write-ups, lab logs and bug-bounty reports are often more convincing
  than a certificate. Make your work visible.
- **Expecting to start as a senior pentester.** Adjacent roles first are normal and valuable; they
  build the foundation.
- **Practising on systems you don't own to "build skills".** Use HTB/THM/VulnHub/your lab/bug bounty
  — all legal. Illegal "practice" ends careers before they start.

## Exercise

1. Pick one entry-level and one hands-on certification that fit your goals. For each, write what it
   tests, its exam format, and what it does and doesn't prove.
2. Start a portfolio: complete one PortSwigger Academy module or one TryHackMe/HTB box and write a
   clear write-up (method, steps, lesson learned) — practising the reporting skill.
3. Draft your own realistic 12-month plan with certs, practice platforms, and (if needed) an
   adjacent role.
4. Explain why a 24-hour practical exam predicts job performance better than a multiple-choice exam,
   using the "format matches the work" idea.
5. List three *legal* ways to practise on real or realistic systems, and explain why illegal
   "practice" is a career-ender (tie back to Level 1 lesson 1).
