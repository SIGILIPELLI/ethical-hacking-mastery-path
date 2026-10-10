---
description: "Service enumeration: turning open ports into named, versioned services and an attack-surface map — banner grabbing, web, SMB, DNS and SNMP."
---

# Service Enumeration

Scanning gave you a list of open ports. Enumeration digs into each one to answer: *what exactly is
running here, which version, how is it configured, and what can I interact with?* It is the phase
where a bare port list becomes a map of the attack surface — and it is where most real findings are
actually discovered. The saying in the field is "enumeration is key," and it is true: time spent
enumerating is rarely wasted, while time spent firing exploits at un-enumerated ports usually is.

Everything here runs against your lab targets.

## The mindset

For every open port, ask:

1. What service and version is this? (Often `-sV` told you; confirm it.)
2. Does this version have known vulnerabilities? (Note them for the CVE lesson.)
3. What can I interact with *without* exploiting anything — banners, listings, anonymous access,
   default content?
4. What does this service reveal about the host and the rest of the network?

A huge amount is available just by *asking politely* — many services volunteer their version,
allow anonymous access, or expose listings by default. Enumeration harvests all of that before any
exploitation is attempted.

## Banner grabbing

Many services announce themselves on connect. You can read the banner by hand with netcat:

```bash
nc -nv 192.168.56.101 21      # connect to FTP; it usually prints its version banner
nc -nv 192.168.56.101 25      # SMTP greets with its software and version
```

Here is a real, reproducible example against a local service — grabbing HTTP response headers,
which reveal the server software:

```bash
curl -I http://127.0.0.1:PORT/
```

against a local web server returns lines like `Server: nginx/1.25.3` or `Server: Apache/2.4.58`,
`X-Powered-By: PHP/8.2`, and so on — each a fact about the stack. (Run it against a web app you
start locally in Level 2; the exact header depends on what you run.)

## Web service enumeration

Web ports (80, 443, 8080, …) are usually the richest. Beyond headers:

- **`robots.txt` and `sitemap.xml`** — often list paths the owner would rather you not visit.
- **Directory and file discovery** — tools like `gobuster`, `ffuf` or `dirb` request many common
  paths from a wordlist to find unlinked pages (`/admin`, `/backup`, `/.git`). Against your own lab:

  ```bash
  gobuster dir -u http://192.168.56.101/ -w /usr/share/wordlists/dirb/common.txt
  ```
- **Technology fingerprinting** — `whatweb http://192.168.56.101/` or browser dev-tools identify
  the CMS, framework and libraries (and their versions).
- **Default credentials and default content** — admin panels, sample apps, setup pages left
  enabled.

## SMB enumeration (Windows and Samba)

SMB (port 445) is a frequent weak point. Enumerate shares, users and the SMB version:

```bash
smbclient -L //192.168.56.101/ -N        # list shares, no password (anonymous)
enum4linux -a 192.168.56.101             # broad SMB/NetBIOS enumeration
nmap --script smb-enum-shares,smb-os-discovery -p445 192.168.56.101
```

Anonymous (null-session) access to shares is a classic finding — it can leak files, usernames and
password-policy information before you authenticate to anything.

## DNS enumeration

If the target runs DNS, try a **zone transfer** (a misconfiguration that dumps every record):

```bash
dig AXFR @192.168.56.101 internal.lab
```

A successful AXFR against a server that shouldn't allow it hands you the whole internal namespace —
every host the zone knows about.

## SNMP enumeration

SNMP (UDP 161) with a default community string (`public`) can leak a startling amount — running
processes, interfaces, installed software, sometimes credentials:

```bash
snmpwalk -v2c -c public 192.168.56.101
```

## Keep it organised

Enumeration generates a lot of detail per host. A good structure per target:

```
Host 192.168.56.101
  21/tcp  ftp   vsftpd <ver>   — anonymous login? [yes/no]  — notes
  22/tcp  ssh   OpenSSH <ver>  — version, banner
  80/tcp  http  <server/ver>   — /admin found, robots.txt lists /backup
  445/tcp smb   Samba <ver>    — null session lists share 'tmp'
```

This table *is* your attack surface, and it feeds straight into vulnerability analysis and the
report.

## How It Actually Works

Why do so many services hand over their version and sometimes their contents for free? Two reasons,
one historical and one structural.

Historically, many protocols were designed in a more trusting era for *interoperability and
debugging*, not secrecy. SMTP, FTP and many others greet a client with a banner identifying the
software and version precisely so that clients and administrators can detect capabilities and
diagnose problems. HTTP's `Server` header exists so caches and clients can adapt. Nobody designing
these protocols treated the version as a secret — and from a pure-security view it arguably isn't;
hiding a version ("security through obscurity") doesn't fix the underlying bug. But for a tester,
that freely-offered version string is the single most valuable fact on the host, because it maps
directly to a list of known vulnerabilities.

Structurally, features like SMB null sessions, SNMP default community strings, DNS zone transfers
and anonymous FTP are **conveniences that default to open**. Null sessions let Windows machines
discover each other; zone transfers let secondary DNS servers sync; `public` is SNMP's shipped
default community. Each is useful in a trusted internal network and dangerous when exposed. The
service isn't "broken" — it is doing exactly what it was configured (often by default) to do. That
is why enumeration finds so much without exploiting anything: you are collecting information the
services are *designed* to give, and the finding is usually that this design-time openness was never
locked down. The fix, almost always, is configuration (disable null sessions, change/disable SNMP
communities, restrict zone transfers, require FTP auth), which is why enumeration findings tend to
be cheap for the client to remediate and valuable to report.

## Common mistakes and pitfalls

- **Enumerating only the "interesting" ports.** The forgotten SNMP service or anonymous FTP is
  often the way in. Enumerate everything open.
- **Stopping at the version string.** The version points to known vulns, but enumeration also finds
  misconfigurations (null sessions, default creds) that no CVE lists.
- **Brute-forcing directories with a giant wordlist against a fragile app** and knocking it over.
  Start small; respect the RoE on load.
- **Ignoring UDP services.** SNMP, the richest enumeration source on some hosts, is UDP-only.
- **Not recording *negative* results.** "FTP anonymous login: denied" is worth noting — it tells
  future-you (and the report) that you checked.
- **Treating version banners as ground truth.** Banners can be edited or misleading; corroborate
  with behaviour where it matters.

## Exercise

1. Start a simple web server locally (e.g. `python3 -m http.server 8000`) and run `curl -I
   http://127.0.0.1:8000/`. Record the `Server` header and explain what it reveals.
2. Against a lab target, grab a banner from at least one text protocol (FTP or SMTP) with `nc`.
   Record the version string.
3. On a lab Windows/Samba target, attempt `smbclient -L //<ip>/ -N`. Did a null session work? Note
   the result either way.
4. Build the per-host enumeration table (as shown above) for one lab target, filling every open
   port with service, version and one note.
5. Explain, using *How It Actually Works*, why a service volunteering its version is not itself a
   vulnerability but is still extremely useful to a tester.
