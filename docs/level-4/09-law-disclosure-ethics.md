---
description: "The law, responsible disclosure and ethics: computer-crime statutes, authorization, coordinated disclosure, and the tester's code of conduct."
---

# Staying Legal: Law, Disclosure & Ethics

The course opened with authorization and the law, and it closes there too — because for an ethical
hacker, legal and ethical discipline is not a constraint on the job; it *is* the job. The techniques
are identical to a criminal's; everything that makes you a professional rather than a felon lives in
how you handle permission, disclosure and the data you touch. This lesson consolidates the legal
landscape, the right way to disclose vulnerabilities, and the ethical code that keeps you — and the
people whose systems you test — safe.

## The law (consolidated)

Computer-crime law centres on **unauthorized access** — not harm (Level 1 lesson 1). The headline
statutes again, because they are the ground you stand on:

- **US — CFAA (18 U.S.C. § 1030):** accessing a computer "without authorization or exceeding
  authorized access."
- **UK — Computer Misuse Act 1990:** s1 unauthorized access, s3 impairing operation; even
  *attempting* is an offence.
- **EU — the Directive on attacks against information systems** (and each member state's
  implementing law).
- **India — IT Act 2000, §43 & §66:** unauthorized access/damage, criminalised when dishonest or
  fraudulent.
- Most countries have an equivalent, and additional laws may apply to the *data* you encounter —
  notably privacy/data-protection law (GDPR in the EU, and others) governing personal data.

The through-line: **authorization is the thing.** Have it, in writing, from someone with authority,
for the specific targets and window — or don't touch the system. There is no learning exemption, no
"I reported it so it's fine," no "it was already exposed."

## The grey zones (and why to avoid them)

- **"The data was public, so accessing it wasn't unauthorized."** Dangerous. Enumerating an API that
  returns other users' records, or incrementing an ID to read data not meant for you, has been
  prosecuted even when no authentication was bypassed. If it isn't yours and you weren't authorized,
  don't.
- **"I found it by accident and just looked a bit more."** The "bit more" is the unauthorized part.
  Stop at discovery; report through a proper channel.
- **Scanning the internet "for research."** Even non-intrusive scanning can breach terms and some
  laws; at best it's a grey zone. Keep research to systems you own, lab targets, and authorized
  programs.
- **Testing a SaaS "you use".** Using a service doesn't authorize testing it; most terms forbid it.
  Check the provider's policy.

When in doubt, the professional answer is **get explicit permission or don't do it**. The asymmetry
is stark: the upside of poking an unauthorized system is negligible; the downside is criminal
liability.

## Responsible / coordinated disclosure

When you find a vulnerability — in authorized work, in a bug-bounty program, or by stumbling on one —
how you disclose it matters ethically and legally.

- **Coordinated Vulnerability Disclosure (CVD):** report privately to the vendor/owner, give them
  reasonable time to fix it, then (by agreement) it may be disclosed publicly. This balances getting
  it fixed against users' safety.
- **Full disclosure** (publishing immediately) pressures vendors but exposes users while unpatched —
  generally discouraged except as a last resort when a vendor refuses to act.
- **A disclosure policy / security.txt:** many organisations publish how to report (a `security.txt`
  file, a bug-bounty program, a security contact). Use it. Its existence is a form of authorization to
  report (not necessarily to test beyond the program's scope).
- **If there's no channel and you found it incidentally:** report carefully through any responsible
  contact, document that you did not exploit or exfiltrate beyond proof, and be aware that some
  organisations react badly even to good-faith reports — which is exactly why *authorized* channels
  (bug bounty, CVD programs) are the safe way to do this.

!!! warning "Good intentions are not authorization"
    Reporting a bug you found by unauthorized testing does not retroactively make the testing legal.
    People have faced charges for "helpful" unauthorized hacking. Test what you're permitted to test;
    disclose through proper channels.

## The ethical code

Beyond the law, professional ethics (echoed in codes like (ISC)²'s and EC-Council's):

- **Only test with authorization**, and stay within scope.
- **Do no harm** — don't damage systems, degrade service, or exfiltrate data beyond proof (Level 4
  lesson 1).
- **Protect confidentiality** — handle discovered data with care; honour NDAs; destroy data per the
  agreement.
- **Be honest** — accurate findings and severities; no fabrication; admit what you didn't test
  (Level 4 lesson 2).
- **Disclose responsibly** — get vulnerabilities fixed without needlessly exposing users.
- **Act in the client's and the public's interest**, not for personal gain beyond your fee/bounty.

These aren't abstract virtues; each maps to a concrete behaviour the course has taught — scope
discipline, minimum-sufficient proof, redaction and data destruction, honest reporting, coordinated
disclosure.

## A worked example (reasoning)

While using a website you legitimately have an account on, you notice that changing a number in a URL
shows another user's data. You are *not* in a bug-bounty program here and have no authorization to
test. The right move: **stop** — do not enumerate other IDs, do not pull data, do not "see how bad it
is." Check for a `security.txt` or a published security contact and report what you observed (one
incidental instance) privately, stating clearly that you did not exploit it further. If the
organisation runs a bug-bounty program covering this asset, report through it within its rules. You
resist the strong temptation to "just confirm the scope of it," because each additional access is
unauthorized and the single observation already lets them find and fix the flaw. Good intentions plus
restraint; not good intentions plus unauthorized digging.

## How It Actually Works

Why does the law hinge on authorization rather than harm or intent, and why is "I reported it" no
defence — isn't that unfair to well-meaning researchers? Return to the fact the course opened with:
the target cannot distinguish a tester's actions from an attacker's, because the packets are
identical and intent lives only in the actor's head. A legal system can't read intent off the wire,
and it can't make "harm" the test either, because the most dangerous unauthorized access often causes
no immediate damage — quietly reading a database harms no system while exposing every customer. So the
law draws the only line it reliably can: *consent*. Did the system's owner permit this access? That's
a fact that can be established (a signed letter, a program policy) or not, independent of what was in
the actor's heart. This is genuinely a hard rule for good-faith researchers, and it's why the field
built *structured* authorization channels — bug-bounty programs, CVD policies, `security.txt`,
contracted engagements — to convert the good-faith impulse into *consented* access. The lesson isn't
"the law is unfair"; it's "the law will not supply the authorization for you, so you must obtain it
through the channels that exist, and where none exists, you must not act."

Responsible disclosure resolves a real tension with the same logic. A found vulnerability creates a
conflict: users are safer the sooner it's *fixed* (argues for telling the vendor), but users are at
risk the sooner it's *publicly known while unpatched* (argues for secrecy). Coordinated disclosure
threads this by reporting privately, allowing a reasonable fix window, and only then disclosing —
maximising the chance of a fix while minimising the window of exposure. It works because it aligns
everyone's incentives around the goal that matters: a fixed vulnerability and protected users, which
is the same goal the entire discipline of ethical hacking serves. Full disclosure and "just looking a
bit more" both optimise something else (pressure, curiosity) at users' expense, which is why the
profession frowns on them. In the end, law, disclosure and ethics all reduce to the course's first
principle seen from three angles: the power to find weaknesses is identical to an attacker's, so the
*only* thing that makes wielding it ethical is permission to act and the discipline to act in the
interest of the people whose systems — and whose data — you touch.

## Common mistakes and pitfalls

- **Believing reporting legitimises unauthorized testing.** It doesn't. People have been charged for
  "helpful" hacking. Authorization first, always.
- **Rationalising access to "public" data.** Enumerating others' records has been prosecuted even
  without an auth bypass. Not yours + not authorized = don't.
- **"Just looking a bit more" after an accidental find.** Stop at discovery; report through a proper
  channel.
- **Full disclosure without giving a fix window.** Exposes users; use coordinated disclosure save as
  a last resort.
- **Ignoring data-protection law.** Personal data you encounter is governed by privacy law too;
  mishandling it is a separate liability.
- **Testing a SaaS because you use it.** Usage isn't authorization; check the provider's policy.
- **Treating ethics as soft.** Every ethical rule here maps to a concrete, career-defining behaviour.

## Exercise

1. Write down the computer-crime statute for your jurisdiction, the section covering unauthorized
   access, and one sentence on what it criminalises.
2. Find a real `security.txt` or vulnerability-disclosure policy for a well-known site. Summarise how
   it says to report and what (if anything) it authorizes.
3. Explain coordinated vs full disclosure, and give one situation where each might be chosen.
4. For the "incidental find" scenario in this lesson, list exactly what you would and would not do,
   and justify each with the law and ethics.
5. In your own words, explain why the law hinges on *authorization* rather than harm or intent, and
   why structured channels (bug bounty, CVD) exist to let good-faith research happen legally.
