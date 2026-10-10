---
description: "Level 1 capstone project: run a full passive-recon, scan and enumeration pass against your own lab range and write it up as a structured discovery report."
---

# Project — Recon & Enumeration of Your Lab Range

This project ties Level 1 together. You will run the early phases of the penetration-testing
lifecycle — reconnaissance, scanning and enumeration — against **your own lab**, and produce a
structured discovery report: a document that lists every host, every open port, every identified
service and version, every notable configuration, and a ranked list of leads for the later levels.
No exploitation yet; the goal is a complete, accurate map.

This mirrors what a real engagement produces at the end of its enumeration phase, and it is the
input to everything that follows. Do it thoroughly — a good map makes the rest of the course
straightforward; a sloppy one makes it guesswork.

## Prerequisites

- Your lab from lesson 2: an attacker VM and at least one (ideally two) target VMs on an isolated
  host-only network. If you only have one target, that is fine for the project.
- A notes structure ready (lessons 6 and 8). Decide now where your evidence lives.

## Scope (define it, then obey it)

Even though you own the lab, write a scope line and stick to it — this is the habit that keeps you
safe on real work:

```
In scope:  <your host-only CIDR, e.g. 192.168.56.0/24>
Out of scope: everything else, including the internet and the host's real LAN
Window: now
```

## Step 1 — Host discovery

Find which hosts are alive:

```bash
nmap -sn <your-host-only-CIDR> -oN discovery.txt
```

Record every responding host. These are your targets.

## Step 2 — Passive recon (adapted for a lab)

A lab has no public footprint, so practise the *technique* against something real: pick a large,
well-known organisation and, using only passive sources, gather its domains, a handful of
subdomains (Certificate Transparency), and the technologies it advertises. Write a half-page recon
note. The point is to rehearse the passive workflow before Level 2/3, where you'll apply it to
in-scope targets. (Keep it passive and lawful — read public data only.)

## Step 3 — Full port scan per host

For each live host:

```bash
nmap -sS -p- -T4 <host> -oA <host>-allports     # all 65535 TCP ports (use -sT if not root)
nmap -sU --top-ports 50 <host> -oA <host>-udp   # a sensible UDP subset
```

Record every open/filtered port and its state.

## Step 4 — Version & default-script scan

For each host, version-detect the open ports and run default scripts:

```bash
nmap -sC -sV -p <comma-separated-open-ports> <host> -oA <host>-sv
```

Record the service and version for every open port.

## Step 5 — Enumerate each service

Go service by service (lesson 8). For each, do at least:

- **Web (80/443/8080/…):** `curl -I`, check `robots.txt`, fingerprint with `whatweb`, run a *small*
  directory wordlist with `gobuster`/`ffuf`. Record server, tech, and any interesting paths.
- **SMB (445):** attempt a null session (`smbclient -L //<ip>/ -N`, `enum4linux -a`). Record shares
  and whether anonymous access worked.
- **FTP (21):** banner-grab; test anonymous login. Record version and access.
- **SSH (22):** record the version banner (no brute-forcing in this project).
- **Any other open service:** banner-grab and note the version.

## Step 6 — Map versions to candidate vulnerabilities

For each identified version, search the NVD and `searchsploit` for known CVEs (lesson 9). Record
candidates — do **not** exploit anything yet. For each candidate note the CVE, its CVSS base score,
and whether a public exploit exists. Mark these as *unvalidated leads*.

## Deliverable: the discovery report

Produce a document with these sections:

1. **Scope & methodology** — what you tested, with which tools, when.
2. **Host inventory** — a table of live hosts.
3. **Per-host service map** — the enumeration table from lesson 8 for each host (port, service,
   version, config notes, access results).
4. **Candidate findings** — the ranked list of version→CVE leads, with CVSS and exploit-availability
   notes, each marked unvalidated.
5. **Next steps** — which leads you would validate first and why (highest severity × most reachable).

A good per-host section looks like:

```
Host 192.168.56.101  (Linux)
  21/tcp  open  ftp    vsftpd <ver>    anonymous login: ALLOWED — lists /pub
  22/tcp  open  ssh    OpenSSH <ver>   banner only
  80/tcp  open  http   <server/ver>    /admin (200), robots.txt -> /backup (403)
  139,445 open  smb    Samba <ver>     null session: shares [tmp, print$]
  3306    open  mysql  <ver>           remote connection permitted (banner)
  Candidate leads:
    - <service> <ver> -> CVE-XXXX-YYYY (CVSS n.n, exploit: yes/no) [unvalidated]
```

## Self-check

Before you call it done:

- Every live host appears in the inventory.
- Every open port on every host has a service and (where obtainable) a version.
- Every service has at least one enumeration action recorded, including negative results.
- Every candidate CVE is marked unvalidated and has a source.
- Someone else could re-run your steps from the methodology section alone.

## How It Actually Works

This project works because enumeration is **cumulative and compounding**: each step narrows
uncertainty and feeds the next, exactly as the lifecycle lesson described. Host discovery converts a
CIDR into a host list; the full port scan converts hosts into ports; version detection converts
ports into named software; service enumeration converts software into configurations and access
facts; CVE mapping converts versions into candidate weaknesses. By the end you have transformed "a
network range I own" into "a ranked list of specific, sourced leads" — and crucially, you did it
*without exploiting anything*, which means the map is complete and the risky part (validation and
exploitation) is now aimed precisely instead of sprayed blindly.

The discipline of recording negative results ("anonymous FTP: denied," "null session: refused")
matters more than it seems: it is what lets the report claim *coverage*. A finding says what's
wrong; the negatives say what you checked and found fine, which is how a client knows the test was
thorough rather than lucky. This is also why the report is assembled continuously rather than
written at the end — every command's saved output is a piece of evidence, and the `-oA` files you
produced here are literally the raw material Level 4's reporting lesson turns into a deliverable.

## Exercise

1. Complete all six steps and produce the full discovery report for your lab.
2. In your "next steps" section, pick the single lead you would validate first. Justify it using
   both severity (CVSS) and reachability — not just the highest score.
3. Count: how many open ports did you find across all hosts, and how many did you successfully
   version-identify? If any resisted identification, note what you'd try next.
4. Review your report against the self-check list and fix any gap.
5. In two sentences, explain why doing zero exploitation in this phase actually makes the later
   exploitation phase faster and safer.
