---
description: "Passive reconnaissance and OSINT: gathering intelligence from public sources without sending the target a packet, and the scope limits that apply."
---

# Passive Reconnaissance & OSINT

Reconnaissance is the first technical phase, and it starts *passive*: gathering everything you can
about a target **without touching the target's systems at all.** No scans, no connections — just
public information. Passive recon is quiet (the target cannot detect it), cheap, and often
astonishingly productive. By the time you send your first packet, you want to already know the
target's domains, likely technologies, and where the interesting systems probably are.

!!! warning "Passive does not mean 'anything goes'"
    Passive recon reads public information, but your **scope and the law still apply**. Querying a
    third party's systems to learn about your client (e.g. aggressively scraping a vendor) can cross
    lines. And remember: the *results* of recon still must stay within the authorized scope when you
    move to active phases. Finding an out-of-scope asset does not make it in scope.

## What you are looking for

- **Domains and subdomains** — the organisation's web presence and the hosts behind it.
- **IP ranges** — the network blocks the organisation owns.
- **Technologies** — web servers, frameworks, CMSes, cloud providers.
- **People** — employees, emails, job titles (relevant to social engineering, Level 3 lesson 9).
- **Leaked data** — credentials in breach dumps, secrets in public code repos, documents with
  metadata.

## Passive sources and techniques

### WHOIS and DNS

WHOIS records tell you who registered a domain and when. DNS records map names to addresses and
reveal mail servers, subdomains and more. These queries hit *registries and DNS servers*, not the
target's own web servers, so they are passive with respect to the target.

```bash
whois example.com            # registration details
dig example.com ANY          # DNS records
dig example.com MX           # mail servers
dig +short example.com       # just the address(es)
```

A concrete, runnable example against a domain that exists for this purpose:

```bash
dig +short example.com
```

`example.com` is reserved by IANA for documentation, so this is safe to run and returns its
documented address(es). On a real engagement you would run it against in-scope domains only.

### Subdomain discovery (passive)

Subdomains often expose staging sites, admin panels and forgotten apps. Passive discovery uses
third-party datasets rather than brute-forcing the target:

- **Certificate Transparency logs** (e.g. via `crt.sh`) — every TLS certificate issued is logged
  publicly, and certificates list the hostnames they cover. Searching CT logs for a domain reveals
  subdomains the organisation requested certs for, without ever querying the target.
- **Search engines** with operators: `site:example.com`, `-www` to exclude the main site,
  `filetype:pdf` to find documents.
- **Passive DNS** databases and OSINT aggregators (SecurityTrails, Shodan, Censys) that have already
  collected this data.

### Google dorking

"Dorking" is using advanced search operators to surface things that are public but not meant to be
found: exposed directories, config files, login pages, documents. Examples of operators (run against
in-scope domains only):

```
site:example.com inurl:admin
site:example.com filetype:xlsx
intitle:"index of" site:example.com
```

### Metadata and code

- **Document metadata** — PDFs, Office files and images embed author names, software versions,
  sometimes internal paths and usernames. Tools like `exiftool` extract it.
- **Public code repositories** — developers accidentally commit API keys, passwords and internal
  hostnames. Searching an organisation's public repos (and historical commits) is a rich source.
- **Breach data** — services that index known breaches tell you which company emails have appeared
  in public dumps (useful for password-spray planning *within an authorized test*).

### Shodan and Censys

Shodan and Censys continuously scan the internet and let you *search the results*. Querying them for
an organisation's IP range shows open ports and banners the scanners already collected — so you
learn what's exposed without scanning the target yourself. This is the passive/active boundary at
its sharpest: *you* send no packets to the target; you query someone else's scan data.

## Organising what you find

Recon produces a lot of scattered facts. Keep them structured from the start — a simple set of
lists (domains, subdomains, IPs, technologies, emails, interesting URLs) that you add to as you go.
This becomes the input to scanning and, later, part of the report's methodology section. Tools like
**Maltego**, **theHarvester** and **Recon-ng** help automate and organise collection, but a
disciplined notes file is the non-negotiable minimum.

## How It Actually Works

Why is so much private-feeling information publicly available? Because the modern internet is built
on **public infrastructure that logs by design**:

- **Certificate Transparency** exists to catch misissued certificates: browsers require that certs
  be logged in public, append-only CT logs, or they won't be trusted. A security feature for the web
  as a whole is, for a tester, a free list of a target's hostnames — including internal-sounding
  ones the organisation got certs for.
- **DNS is a public lookup system.** It has to be queryable by anyone for the internet to work, so
  mail servers, subdomains and address records are answerable questions by design.
- **WHOIS** was built as a public registry of who owns what.
- **Internet-wide scanners** (Shodan, Censys) simply do once, continuously, what you would do
  actively — and publish the results — so the exposure of a misconfigured service becomes searchable
  history.

The deeper point: information leaks not because any single system is misconfigured, but because the
*ecosystem* is designed for transparency and interoperability, and that transparency aggregates. Any
one fact (a cert here, a commit there, a job posting mentioning "we use Okta and AWS") is harmless;
combined, they sketch the target's attack surface before you touch it. This is why passive recon is
both powerful and quiet — you are reading the exhaust the organisation and its ecosystem emit as a
matter of normal operation, and reading it costs the target nothing and reveals nothing about you.

## Common mistakes and pitfalls

- **Treating passive recon as optional.** Testers who skip it waste active scanning on the wrong
  targets and miss the forgotten staging box that CT logs would have handed them.
- **Letting recon drift out of scope.** A subdomain you find on a shared hosting provider may
  resolve to infrastructure your client doesn't own. In scope is what the authorization says, not
  what you can find.
- **Scraping aggressively enough to be "active."** Hammering a third-party service to enumerate your
  target can itself be unauthorized activity against that third party.
- **Not recording sources.** "I found an exposed admin panel" is worthless in a report without the
  URL, timestamp and how you found it.
- **Confusing Shodan results with current reality.** Scanner data has a date; a port Shodan saw open
  last month may be closed now. Treat it as a lead, confirm later.

## Exercise

1. Run `whois example.com` and `dig +short example.com`. Record the registrar and the address(es)
   returned. (These are safe, documentation-reserved values.)
2. Visit a Certificate Transparency search (e.g. `crt.sh`) and look up a large, well-known
   organisation's primary domain. List five subdomains it reveals and note which sound like
   non-production systems.
3. Write three Google dork queries you would use to find exposed documents, admin pages and
   directory listings for an in-scope domain. Explain what each operator does.
4. Start a structured recon notes file with sections for domains, subdomains, IPs, technologies and
   emails. You will fill it during the Level 1 project.
5. Explain, using the *How It Actually Works* section, why Certificate Transparency — a defensive
   feature — is useful to an attacker, and why that is not a flaw in CT.
