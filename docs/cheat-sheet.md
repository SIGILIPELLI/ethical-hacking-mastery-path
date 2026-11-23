---
title: "Ethical Hacking Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Ethical Hacking cheat sheet: 15 quick-reference tables from every level of the Ethical Hacking Mastery Path, each linked to its full lesson."
---

# Ethical Hacking Cheat Sheet

Quick-reference tables from **15 lessons** of the free Ethical Hacking Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Foundations & Legal Lab

### [What Ethical Hacking Is — Authorization, Scope &amp; the Law](level-1/01-what-ethical-hacking-is.md)

*The three things that make it ethical*

| Element | Question it answers | Where it lives |
|---------|--------------------|-----------------|
| **Authorization** | Who said I can do this? | A signed contract / authorization letter |
| **Scope** | What exactly am I allowed to touch? | The statement of work / scope document |
| **Rules of Engagement (RoE)** | How, when and how hard? | The RoE document |

### [Building a Legal, Isolated Practice Lab](level-1/02-building-a-lab.md)

*The one rule that keeps it legal: isolation*

| Mode | VM can reach… | Use for the lab? |
|------|---------------|------------------|
| **NAT** | the internet, via the host | Only briefly, to install updates |
| **Bridged** | your real LAN and its devices | **No** — this puts the VM on your home network |
| **Host-only** | the host and other host-only VMs | **Yes** — isolated from the internet and LAN |
| **Internal** | only other VMs on the same internal net | **Yes** — even more isolated (no host) |

### [The Penetration Testing Lifecycle](level-1/03-pentest-lifecycle.md)

*A worked example (lab)*

| Phase | What you do | Output |
|-------|-------------|--------|
| Pre-engagement | (In the lab, you own it — but note scope: just this one host.) | Target: `192.168.56.101` only |
| Recon | Nothing to OSINT in a lab; you'd gather public info for a real target | — |
| Scanning | `nmap -sV 192.168.56.101` | A list of open ports and service versions |
| Enumeration | Dig into each service — banner, shares, web paths | Named services, versions, likely weak points |
| Exploitation | Use a known-vuln service to get a shell | A foothold |
| Post-exploitation | Check privileges, look for other hosts, sensitive files | Impact assessment |
| Reporting | Write each finding with reproduction and fix | The deliverable |

### [Networking Through a Tester's Lens](level-1/05-networking-for-testers.md)

*The layers, briefly*

| Layer (TCP/IP model) | Unit | Examples | What it gives a tester |
|----------------------|------|----------|------------------------|
| Application | data | HTTP, DNS, SSH, SMB | The actual service to attack |
| Transport | segment | TCP, UDP + **ports** | Which services are reachable |
| Internet | packet | IP, ICMP | Which hosts exist and are up |
| Link | frame | Ethernet, ARP, Wi-Fi | Local-network attacks, MAC addresses |

### [Active Scanning with Nmap](level-1/07-active-scanning-nmap.md)

*Port states*

| State | Meaning |
|-------|---------|
| **open** | A service is listening and accepting connections |
| **closed** | The host answered, but nothing is listening there |
| **filtered** | No useful answer — a firewall likely dropped the probe |
| **open\|filtered** | Can't tell open from filtered (common on UDP) |

### [Vulnerabilities, CVE &amp; CVSS](level-1/09-vulnerabilities-cve-cvss.md)

*CVSS: how severity is scored*

| Metric | Asks | Example values |
|--------|------|----------------|
| Attack Vector (AV) | How close must the attacker be? | Network / Adjacent / Local / Physical |
| Attack Complexity (AC) | How hard to pull off reliably? | Low / High |
| Privileges Required (PR) | What access is needed first? | None / Low / High |
| User Interaction (UI) | Must a victim do something? | None / Required |
| Scope (S) | Can it affect components beyond the vulnerable one? | Unchanged / Changed |
| Confidentiality / Integrity / Availability (C/I/A) | Impact on each? | None / Low / High |

## Level 2 · Web Application Testing

### [Cross-Site Scripting (XSS)](level-2/04-xss.md)

*The three types*

| Type | Where the payload lives | Trigger |
|------|-------------------------|---------|
| **Reflected** | In the request (e.g. a URL parameter), echoed into the immediate response | Victim clicks a crafted link |
| **Stored** | Saved on the server (a comment, profile, message) and served to others | Victim views the page containing it |
| **DOM-based** | Never leaves the browser; client-side JS writes attacker input into the page | Victim loads a page whose JS mishandles input |

### [Misconfiguration &amp; Vulnerable Components](level-2/08-misconfiguration-components.md)

*Missing security headers*

| Header | Protects against |
|--------|-------------------|
| `Content-Security-Policy` | XSS, data injection (restricts script sources) |
| `Strict-Transport-Security` | Protocol downgrade / SSL stripping |
| `X-Content-Type-Options: nosniff` | MIME-sniffing attacks |
| `X-Frame-Options` / CSP `frame-ancestors` | Clickjacking |
| `Set-Cookie` flags (`HttpOnly`, `Secure`, `SameSite`) | Cookie theft / CSRF |

## Level 3 · Infrastructure & Network Testing

### [Vulnerability Scanning &amp; Validation](level-3/01-vuln-scanning-validation.md)

*A worked example (the reasoning)*

| Scanner says | Validation | Verdict |
|--------------|-----------|---------|
| "Service X vX.Y — CVE-A, Critical" | Banner confirms vX.Y; feature enabled; lab exploit from Exploit-DB gets a shell | **Real — Critical.** Document with evidence. |
| "TLS supports an old protocol — Medium" | `testssl.sh` confirms the old protocol is offered | **Real — Medium.** Config finding. |
| "Service Z — CVE-C, High" | Banner says vulnerable version, but the vulnerable module is not loaded; exploit fails | **False positive.** Note it as tested-and-not-exploitable. |

### [Exploitation Fundamentals &amp; Metasploit](level-3/02-exploitation-metasploit.md)

*The Metasploit Framework*

| Module type | What it is |
|-------------|------------|
| **exploit** | Triggers a specific vulnerability |
| **payload** | What runs on success (shells, Meterpreter, single commands) |
| **auxiliary** | Scanners, fuzzers, non-exploit tools |
| **post** | Post-exploitation actions once you have a session |
| **encoder / nop** | Transform payloads (e.g. to avoid bad bytes) |

## Level 4 · Professional Practice

### [Writing the Penetration Test Report](level-4/02-writing-the-report.md)

*Who reads it, and what they need*

| Reader | Wants | Section that serves them |
|--------|-------|--------------------------|
| Executives / management | Overall risk, business impact, "how bad, what now" | Executive summary |
| Security / engineering | Exactly what's wrong, how to reproduce, how to fix | Detailed findings |
| Compliance / auditors | Scope, methodology, evidence of a thorough test | Scope & methodology |
| Future testers (retest) | What was found, so they can verify fixes | Findings + reproduction |

### [Threat Modeling &amp; Attack Surface Mapping](level-4/04-threat-modeling.md)

*STRIDE — a threat taxonomy*

| Threat | Violates | Example |
|--------|----------|---------|
| **S**poofing | Authentication | Pretending to be another user/system |
| **T**ampering | Integrity | Modifying data in transit or at rest |
| **R**epudiation | Non-repudiation | Denying an action with no audit trail |
| **I**nformation disclosure | Confidentiality | Leaking data to the unauthorized |
| **D**enial of service | Availability | Making a resource unavailable |
| **E**levation of privilege | Authorization | Gaining capabilities you shouldn't have |

### [Cloud Penetration Testing Basics](level-4/05-cloud-pentesting.md)

*The shared-responsibility model*

| Layer | IaaS (VMs) | PaaS (managed app) | SaaS (finished app) |
|-------|-----------|--------------------|--------------------|
| Physical / hardware | Provider | Provider | Provider |
| Hypervisor / host OS | Provider | Provider | Provider |
| Guest OS / runtime | **Customer** | Provider | Provider |
| App / code | **Customer** | **Customer** | Provider |
| **Configuration & IAM** | **Customer** | **Customer** | **Customer** |
| **Data** | **Customer** | **Customer** | **Customer** |

### [Mobile Application Testing Basics](level-4/06-mobile-testing.md)

*The two platforms*

| Aspect | Android | iOS |
|--------|---------|-----|
| Package | APK/AAB | IPA |
| Language | Kotlin/Java (+ native) | Swift/Objective-C |
| Local storage | SharedPreferences, SQLite, files | UserDefaults, Keychain, Core Data, files |
| IPC | Intents, content providers, services | URL schemes, universal links, extensions |
| Test device | Emulator or rooted device | Simulator (limited) or jailbroken device |

### [Certifications &amp; Career Path](level-4/08-certifications-career.md)

*The major certifications*

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

---

Go deeper with the full [Ethical Hacking Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
