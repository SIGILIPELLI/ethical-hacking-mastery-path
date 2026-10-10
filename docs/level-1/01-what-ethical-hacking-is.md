---
description: "What separates ethical hacking from a crime: written authorization, defined scope, rules of engagement, and the computer-crime laws every tester must know."
---

# What Ethical Hacking Is — Authorization, Scope & the Law

Ethical hacking is using the same skills, tools and mindset as an attacker to find security
weaknesses — **with permission, inside an agreed boundary, for the purpose of getting them
fixed.** The techniques are identical to a criminal's. The only things that make the work lawful
and professional are authorization, scope, and intent. Lose any one of those and you are not a
penetration tester; you are committing a computer crime.

This first lesson is not a warm-up you can skip to get to the "real" content. The ability to
answer *"am I allowed to do this, to this target, right now?"* is the single most important skill
in the field, and it is the thing that keeps you out of prison and in work.

## The three things that make it ethical

| Element | Question it answers | Where it lives |
|---------|--------------------|-----------------|
| **Authorization** | Who said I can do this? | A signed contract / authorization letter |
| **Scope** | What exactly am I allowed to touch? | The statement of work / scope document |
| **Rules of Engagement (RoE)** | How, when and how hard? | The RoE document |

Without a signed authorization, every other detail is irrelevant. A verbal "sure, go ahead" from
someone in a hallway is not authorization — the person may not have the authority to grant it, and
you have nothing in writing if it goes wrong. Professionals do not touch a target until the
paperwork exists.

### Authorization

Authorization is written permission from someone who actually owns the system or has the authority
to allow testing of it. For an external client this is a signed contract plus an **authorization
letter** (sometimes called a "get out of jail free" letter) that you carry during the engagement.
It names:

- the testing company and the specific individuals authorized to test;
- the client entity granting permission and the signatory's authority to grant it;
- the exact targets (IP ranges, domains, applications);
- the dates and times testing is allowed;
- an emergency contact who can be reached 24/7.

The reason the letter names *you specifically* is that authorization is not transferable. If a
colleague is not on the letter, they are not authorized, even if they work at the same firm.

### Scope

Scope is the precise list of what is in bounds. It is usually expressed as:

- **In scope:** `10.10.10.0/24`, `app.example-client.com`, the staging API.
- **Out of scope:** everything else — explicitly including third-party services the client uses
  but does not own (their payment processor, their email provider, their cloud control plane).

Scope exists because a client can only authorize testing of systems *they* own. Their SaaS vendors,
their bank, their upstream ISP — those belong to someone else, and the client cannot grant you
permission to test them. Attacking an out-of-scope third party is unauthorized access even though
the client hired you.

### Rules of Engagement

RoE covers the *how*: permitted techniques, forbidden techniques (often no denial-of-service, no
social engineering unless separately agreed, no testing during business hours), data-handling
rules (you will see real data — how must you store and destroy it?), and what to do if you find
something critical mid-test or stumble on evidence of a prior breach.

## The law you must know

Computer-crime law criminalises **unauthorized access** — not "damage", not "theft", just access
you were not permitted. This is why authorization is everything.

!!! danger "These are real criminal statutes"
    - **United States — Computer Fraud and Abuse Act (CFAA), 18 U.S.C. § 1030.** Prohibits
      accessing a computer "without authorization or exceeding authorized access." Penalties scale
      to years of imprisonment.
    - **United Kingdom — Computer Misuse Act 1990.** Section 1: unauthorized access. Section 3:
      unauthorized acts impairing operation. Even *attempting* unauthorized access is an offence.
    - **India — Information Technology Act, 2000, §43 and §66.** §43 imposes liability for
      unauthorized access, downloading, or damage; §66 makes dishonest or fraudulent §43 acts a
      criminal offence punishable by imprisonment.
    - Most other countries have an equivalent. Scanning, let alone exploiting, a system you do not
      own or have permission to test can be a crime by itself.

There is no learning exemption. "I was only practising" is not a defence. The entire reason Level 1
teaches you to build an isolated lab (next lesson) is so that you can practise every technique in
this course against systems *you own*, where none of these statutes are engaged.

## How It Actually Works

Why does the law hinge on *authorization* rather than *harm*? Because from the target's side, a
port scan from a legitimate tester and a port scan from an attacker are **byte-for-byte identical**.
The packets are the same; the only difference is permission, which lives entirely outside the
network traffic. The system being probed cannot tell intent from the wire. So the legal system
cannot use "did it cause damage" as the line — a scan that causes no damage is still unauthorized
access if you had no permission. The line has to be drawn at consent, and consent has to be
provable, which is why it must be in writing and signed by someone with authority.

This also explains the structure of a professional engagement. Everything — the authorization
letter, the tightly-defined scope, the emergency contact, the data-handling rules — exists to
convert an activity that is otherwise criminal into one that is contractually permitted and
bounded. The paperwork is not bureaucracy; it is the mechanism that makes the work legal. A tester
who treats scoping as an annoyance to rush through has misunderstood the job.

Think of it like a locksmith hired to test whether your locks can be picked. Picking a stranger's
front door is burglary; picking the door of the client who hired you, during the agreed window, is
a service. The physical act is the same. The contract is what changes its legal nature — and if the
locksmith wanders next door "to check that lock too," they have committed a crime regardless of
intent.

## Common mistakes and pitfalls

- **Scope creep mid-test.** You find that an in-scope server trusts an out-of-scope one and it
  would be *so easy* to pivot. Stop. That host is not authorized. Note it as a finding ("in-scope
  host X trusts out-of-scope host Y") and move on. Pivoting to it is unauthorized access.
- **Assuming a subdomain is in scope.** `app.example.com` being in scope does not put
  `mail.example.com` in scope, and definitely not `example-cdn.net`. Test only what is listed.
- **Treating a verbal OK as authorization.** Get it in writing, signed by someone who can grant it.
- **Testing "your own" account on a SaaS without checking its rules.** Many providers forbid
  testing even your own tenant because it shares infrastructure with others. More on this in the
  cloud lesson.
- **Keeping client data after the engagement.** Screenshots of real customer records, cracked
  password lists — these must be handled and destroyed per the RoE. Hoarding them is both an ethics
  and often a legal breach.
- **Forgetting that authorization is time-bounded.** Testing before the start date or after the
  end date is outside the authorization even if the target hasn't changed.

## Exercise

1. Write a one-paragraph plain-English answer to: *"A friend says their company's website is
   insecure and asks you to 'just take a quick look.' What do you need before you touch it, and
   from whom?"* Name the specific document and who must sign it.
2. Draft a minimal authorization-letter checklist: list the six items (from the Authorization
   section) that must appear before you would begin a test. For each, write one sentence on what
   goes wrong if it is missing.
3. Look up the computer-crime statute for the country you live in. Write down its name, the section
   that covers unauthorized access, and one sentence on what it prohibits. (If you are in the US,
   UK or India, use the ones above and find the full text.)
4. Explain, in your own words, why "it caused no damage" is not a legal defence to unauthorized
   scanning. Tie your answer back to the *How It Actually Works* section.
