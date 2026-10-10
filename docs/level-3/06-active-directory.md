---
description: "Active Directory attack paths: how a domain is enumerated and why it falls — Kerberos, NTLM hashes, Kerberoasting, pass-the-hash and BloodHound."
---

# Active Directory Attack Paths

Most corporate Windows networks are **Active Directory (AD)** domains: a central directory of users,
computers, groups and policies, with single sign-on across the whole estate. That centralisation is
a huge convenience — and a huge target, because compromising the domain usually means compromising
*everything*. This lesson explains how AD is enumerated and why it falls: the authentication
protocols, the characteristic attacks, and the tooling. It is conceptual plus lab; build a small AD
lab (a Domain Controller and a couple of joined machines) to practise.

## What Active Directory is

- A **domain** is a collection of objects (users, computers, groups) managed centrally.
- A **Domain Controller (DC)** hosts the directory and authenticates everyone.
- **Kerberos** is the primary authentication protocol; **NTLM** is the older fallback still widely
  present.
- **Domain Admins** effectively own the domain; the DC's database (**NTDS.dit**) holds every account's
  password hash.

The attacker's goal is usually Domain Admin, or direct access to NTDS.dit — either equals "own the
domain."

## Enumeration

Once you have *any* domain foothold (a single user's credentials or a shell on a joined machine), AD
is extraordinarily enumerable because members are *meant* to query the directory:

- **Users, groups, computers, policies** — via LDAP queries, `net` commands, PowerView, or
  `ldapsearch`.
- **BloodHound** — collects relationships (who is admin where, who can reset whose password, group
  nesting) and renders **attack paths** as a graph. Its signature output is a shortest path from
  "the account I control" to "Domain Admin." BloodHound turned AD attack-path analysis from manual
  slog into a query.
- **Shares and sessions** — find where privileged users are logged in (so you can target those
  machines) and what's on accessible shares.

## Why AD falls — the characteristic attacks

### Kerberoasting

Service accounts in AD are identified by **Service Principal Names (SPNs)**. Any authenticated user
can *request* a Kerberos service ticket for an SPN, and part of that ticket is encrypted with the
service account's password hash. So an ordinary user can request tickets for service accounts, take
them offline, and **crack** them (Level 3 lesson 3) — service accounts often have old, weak,
non-expiring passwords. No special privilege needed to *request*; the weakness is crackable service
passwords.

### AS-REP roasting

Accounts with "Kerberos pre-authentication" disabled let anyone request authentication data that is
encrypted with the user's password hash — again crackable offline, again needing no credentials to
request.

### Pass-the-Hash (PtH)

NTLM authentication proves you know the password by using its **hash**, not the password itself. So
if you *capture a hash* (from a local SAM, from memory, from NTDS), you can authenticate as that user
**without ever cracking it** — you pass the hash directly. This is why "I got the hash but can't crack
it" is often irrelevant: for NTLM, the hash *is* the credential.

### Pass-the-Ticket / Overpass-the-Hash

Similar idea for Kerberos: steal or forge Kerberos tickets and reuse them. The extreme cases are
forged tickets — a **Golden Ticket** (forged Ticket-Granting Ticket using the domain's `krbtgt`
account hash, granting arbitrary access) and a **Silver Ticket** (forged service ticket). These
require already-high privilege (the `krbtgt` hash means you've essentially already won) and are more
about persistence than initial escalation.

### Credential dumping & lateral movement

With local admin on one machine, dump cached credentials and hashes from memory (the classic tool is
**Mimikatz**). Reuse them (PtH / pass-the-ticket) to move to the next machine, repeating until you
reach a Domain Admin's session or the DC. This chain — foothold → dump → reuse → repeat — is how a
single compromised workstation becomes a domain takeover.

## A worked example (reasoning)

In a lab domain you obtain one low-privileged user's credentials (via phishing simulation, a web app,
or a password spray). You run BloodHound's collector as that user; the graph shows a path: your user
is in a group that can reset a service account's password, that service account is local admin on a
server where a Domain Admin has an active session. You Kerberoast to crack a service account (or
follow the BloodHound path), gain local admin on the right server, dump the Domain Admin's
credentials from memory, and pass them to the DC. Each hop is documented with the technique, the
evidence, and the misconfiguration that allowed it. The report's value is the *path* — "these five
ordinary-looking permissions chain into domain compromise" — plus the specific fixes to break it.

## The defences (what breaks the paths)

- **Strong, long, rotated service-account passwords** (or **group Managed Service Accounts**,
  gMSAs) — defeats Kerberoasting by making the crackable hash uncrackable.
- **Enable Kerberos pre-authentication** everywhere — kills AS-REP roasting.
- **Tiered administration / least privilege** — don't let Domain Admins log on to ordinary
  workstations (so their credentials aren't in reach of a workstation compromise); separate admin
  accounts.
- **LAPS** (per-machine random local-admin passwords) — breaks local-admin password reuse and thus
  pass-the-hash lateral movement.
- **Credential Guard** and reduced credential caching — make dumping from memory harder.
- **Monitor** for Kerberoast-style ticket requests, anomalous logons, and run BloodHound *yourself*
  to find and cut the paths before an attacker does.

## How It Actually Works

Why does a single low-privileged foothold so often cascade into total domain compromise, when no
individual AD feature is "broken"? Because AD's design goals — central authentication, single
sign-on, and rich delegation — each create a form of *transitive trust*, and attack paths are just
those trusts chained together. Single sign-on means credentials for the whole domain end up cached in
the memory of machines all over the estate; convenient for users, but it means compromising one
machine where a privileged user logged in yields that user's credentials. Delegation and the
fine-grained permission model mean "user A can reset user B's password," "group C is admin on server
D" — each individually reasonable, but BloodHound's insight was that these relationships form a
*graph*, and an attacker only needs *one path* through it from any controlled node to Domain Admin.
The defenders see a thousand sensible permissions; the attacker sees the shortest path. Centralisation
turns "compromise everything" into "find one route."

The authentication protocols explain the specific attacks. Kerberos was designed so that any
authenticated principal can request a service ticket — necessary for SSO to work — and those tickets
are encrypted with the service account's key; the designers assumed service passwords would be
strong, so "anyone can request a ticket they can try to crack offline" wasn't seen as a hole.
Kerberoasting is the gap between that assumption and reality (weak service passwords). NTLM's
pass-the-hash works because NTLM was built as a *challenge-response over the hash* — proving you hold
the hash *is* the authentication, by protocol design, so the hash is a password-equivalent secret
whether or not you can reverse it. That's not an implementation bug; it's the protocol's model, which
is exactly why mitigations focus on *not letting hashes be captured* (Credential Guard, reduced
caching, tiering) rather than on "fixing" NTLM. Across all of it, the theme is that AD's power and
convenience are inseparable from its blast radius: the same transitive trust that lets an admin
manage ten thousand machines from one console lets an attacker reach ten thousand machines from one
compromised credential. The defences all work by *breaking transitivity* — tiering so privileged
credentials never land on low-trust machines, LAPS so one local password doesn't unlock the next box,
gMSAs so service hashes can't be cracked — cutting the graph so no single path survives.

## Common mistakes and pitfalls

- **Trying to *crack* every hash.** For NTLM, pass-the-hash means you often don't need to. Reuse it.
- **Manual enumeration when BloodHound exists.** You'll miss the non-obvious chained path a graph
  finds instantly. (Understand the relationships it shows, though — don't just click "shortest
  path".)
- **Going loud with Golden/Silver tickets early.** Those need high privilege you don't have yet;
  they're late-stage persistence, not initial escalation.
- **Forgetting that service accounts are the soft underbelly.** Old, weak, over-privileged service
  passwords are the most reliable AD win via Kerberoasting.
- **Running AD attacks outside a lab/authorised scope.** Domain attacks are extremely high-impact;
  the authorization bar is correspondingly high.
- **Reporting individual findings without the path.** The value is the *chain*; show how ordinary
  permissions combine, and which single fix breaks the chain most cheaply.

## Exercise

1. Build a minimal AD lab: a Domain Controller and two joined machines, with a couple of users and a
   service account (give the service account a weak password on purpose).
2. From a low-priv user, run BloodHound's collector and find a path to Domain Admin. Describe the
   path in words.
3. Perform a Kerberoast against your lab: request a service ticket for the SPN, extract it, and crack
   the weak service password offline. Record the steps.
4. Explain pass-the-hash in your own words — why capturing an NTLM hash can be enough to authenticate
   without cracking it — and name the defence that most directly limits its lateral spread.
5. Using the "transitive trust / graph" idea, explain why one compromised workstation can lead to
   domain takeover, and pick the single defence you think breaks the most paths.
