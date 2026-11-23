---
title: "Ethical Hacking Interview Questions and Answers (179 Questions)"
description: "179 Ethical Hacking interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Ethical Hacking Mastery Path."
---

# Ethical Hacking Interview Questions and Answers

**179 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Ethical Hacking Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Foundations & Legal Lab

### [What Ethical Hacking Is — Authorization, Scope &amp; the Law](level-1/01-what-ethical-hacking-is.md)

**Q1.** What makes ethical hacking 'ethical' and lawful?

**Answer:** Written permission, an agreed scope and the purpose of getting weaknesses fixed

The techniques are the same as a criminal's.

**Q2.** You find a vulnerable server that is not on your scope list. What do you do?

**Answer:** Stop and ask the client; testing out-of-scope systems is unauthorized

Scope is a legal boundary.

**Q3.** Who may you test in this course?

**Answer:** Only systems you own or have written authorization to test

Practice in your own lab.

**Q4.** What do ethical hackers deliver?

**Answer:** Findings with evidence, risk ratings and remediation advice

The goal is fixing.

### [Building a Legal, Isolated Practice Lab](level-1/02-building-a-lab.md)

**Q5.** What is a home lab?

**Answer:** Virtual machines on your own computer on an isolated private network

Cannot reach the internet or your real network.

**Q6.** Why isolate the lab network?

**Answer:** Vulnerable machines must not be reachable by others or expose your real devices

Host-only networks.

**Q7.** Which targets are suitable for a lab?

**Answer:** Deliberately vulnerable VMs like Metasploitable or DVWA/Juice Shop

Legal practice targets.

**Q8.** Why snapshot VMs before testing?

**Answer:** So you can restore a clean state after breaking things

Repeatable practice.

### [The Penetration Testing Lifecycle](level-1/03-pentest-lifecycle.md)

**Q9.** What is the typical pentest lifecycle order?

**Answer:** Scoping, reconnaissance, scanning/enumeration, exploitation, post-exploitation, reporting

Phases feed each other.

**Q10.** Why follow a repeatable process?

**Answer:** Thorough, repeatable coverage that another tester could reproduce

Not random poking.

**Q11.** What phase produces the deliverable the client keeps?

**Answer:** Reporting

Findings and fixes.

**Q12.** What should happen before any testing starts?

**Answer:** Agree scope and rules of engagement in writing

Authorization first.

### [The Linux Command Line You'll Actually Use](level-1/04-linux-command-line.md)

**Q13.** Which command searches text in files recursively?

**Answer:** grep -r 'pattern' .

Filter tool output.

**Q14.** What does this pipeline print?

```python
from collections import Counter
lines='80 open\n22 open\n80 open\n443 closed'.split('\n')
print(Counter(l.split()[0] for l in lines).most_common(1))
```

**Answer:** [('80', 2)]

Like `sort | uniq -c | sort -rn`. Verified by running it.

**Q15.** What does `chmod 600 key` do?

**Answer:** Only the owner can read and write the file

Private key hygiene.

**Q16.** Why learn the shell?

**Answer:** Most security tools run on it and emit text you filter and combine

Pipes and redirects.

### [Networking Through a Tester's Lens](level-1/05-networking-for-testers.md)

**Q17.** What does a TCP three-way handshake look like?

**Answer:** SYN, SYN-ACK, ACK

Scanners use it to find open ports.

**Q18.** How many addresses (including network and broadcast) are in 192.168.0.0/26?

```python
import ipaddress
print(ipaddress.ip_network('192.168.0.0/26').num_addresses)
```

**Answer:** 64

2^(32-26). Verified by running it.

**Q19.** What response suggests a closed TCP port?

**Answer:** A RST packet

Open ports answer SYN-ACK.

**Q20.** Why do testers need to understand networking?

**Answer:** Scanning and exploitation are conversations over the network; you must interpret responses

Know what tools do.

### [Passive Reconnaissance &amp; OSINT](level-1/06-passive-recon-osint.md)

**Q21.** What is passive reconnaissance?

**Answer:** Gathering public information without touching the target's systems

Quiet and undetectable.

**Q22.** Which is an OSINT source?

**Answer:** Search engines, WHOIS, certificate transparency and public repos

Public data.

**Q23.** Why is OSINT valuable?

**Answer:** It reveals domains, staff, technologies and leaked data cheaply and quietly

Informs later phases.

**Q24.** What should you not do during passive recon?

**Answer:** Send traffic to the target's hosts

That would be active.

### [Active Scanning with Nmap](level-1/07-active-scanning-nmap.md)

**Q25.** What does a basic nmap SYN scan option -sS do?

**Answer:** Half-open scan sending SYN and reading the reply without completing the handshake

Stealthier than connect scans, needs privileges.

**Q26.** What does `-sV` do?

**Answer:** Detects service versions on open ports

Version detection.

**Q27.** What does `-p-` scan?

**Answer:** All 65,535 TCP ports

Takes longer.

**Q28.** Why scan only authorized targets?

**Answer:** Unauthorized scanning can be illegal and disruptive

Scope.

### [Service Enumeration](level-1/08-service-enumeration.md)

**Q29.** What does enumeration add to a port list?

**Answer:** Exact services, versions, configurations and interactable resources

Where findings come from.

**Q30.** What does a service banner reveal?

**Answer:** The software and often version running on a port

Compare with known CVEs.

**Q31.** What does anonymous FTP access indicate?

**Answer:** Possible exposure of files without credentials: a configuration weakness

Report it.

**Q32.** Why enumerate each service separately?

**Answer:** Each protocol exposes different information and risks

SMB, HTTP, SSH.

### [Vulnerabilities, CVE &amp; CVSS](level-1/09-vulnerabilities-cve-cvss.md)

**Q33.** What does CVE identify?

**Answer:** A specific publicly disclosed vulnerability

CVE-YYYY-NNNN.

**Q34.** What does CVSS provide?

**Answer:** A 0-10 severity score for vulnerabilities

Context still matters.

**Q35.** What is a CWE?

**Answer:** A category of software weakness (e.g. CWE-89 SQL injection)

Weakness types.

**Q36.** A CVSS base score of 9.8 falls into which severity band?

```python
s=9.8
print('Critical' if s>=9 else 'High' if s>=7 else 'Medium' if s>=4 else 'Low')
```

**Answer:** Critical

9.0-10.0 is Critical. Verified by running it.

### [Project — Recon &amp; Enumeration of Your Lab Range](level-1/10-project-recon-enumeration.md)

**Q37.** What does the recon project produce?

**Answer:** A structured discovery report of hosts, ports, services and candidate vulnerabilities

Level 1 deliverable.

**Q38.** Where do you run it?

**Answer:** Against your own isolated lab

Legal.

**Q39.** What should each finding contain?

**Answer:** Evidence, affected host/service and a risk rating

Traceable.

**Q40.** How do you prioritize follow-up?

**Answer:** By severity, exposure and exploitability

Risk-based.

## Level 2 · Web Application Testing

### [How Web Apps Work &amp; the Testing Mindset](level-2/01-how-web-apps-work.md)

**Q41.** What is the web trust boundary?

**Answer:** Everything the client sends can be changed by the client, so the server must validate

Never trust the client.

**Q42.** Why is client-side validation not a security control?

**Answer:** Attackers bypass the browser and send requests directly

Repeat on the server.

**Q43.** What does HTTP being stateless imply?

**Answer:** Sessions are built with cookies/tokens that must be protected

Session management.

**Q44.** Which part of a request can an attacker modify?

**Answer:** All of it: URL, headers, cookies and body

Use a proxy.

### [Using an Intercepting Proxy](level-2/02-intercepting-proxy.md)

**Q45.** What does an intercepting proxy do?

**Answer:** Sits between browser and server so you can read and modify requests

Burp or ZAP.

**Q46.** Why configure the browser to trust the proxy's CA?

**Answer:** To inspect HTTPS traffic in your lab

Only on your test browser.

**Q47.** What does the repeater feature do?

**Answer:** Re-sends a modified request to observe responses

Manual testing.

**Q48.** What must you ensure with proxy testing?

**Answer:** Only target systems you're authorized to test

Scope.

### [Injection Flaws &amp; SQL Injection](level-2/03-sql-injection.md)

**Q49.** An unsafe query built by concatenation vs a parameterized one. What are the counts?

```python
import sqlite3
c=sqlite3.connect(':memory:'); c.execute('create table u(n)'); c.executemany('insert into u values(?)',[('a',),('b',)])
v="x' OR '1'='1"
bad=c.execute("select count(*) from u where n='"+v+"'").fetchone()[0]
good=c.execute('select count(*) from u where n=?',(v,)).fetchone()[0]
print(bad,good)
```

**Answer:** 2 0

Concatenation lets input change the query; parameters keep it data (demonstrated on a local in-memory DB). Verified by running it.

**Q50.** What is the primary defense against SQL injection?

**Answer:** Parameterized queries / prepared statements

Plus least-privilege DB accounts.

**Q51.** Why is a WAF not a complete fix?

**Answer:** It can be bypassed; fix the code

Defense in depth.

**Q52.** What impact can SQL injection have?

**Answer:** Reading, modifying or deleting data and sometimes server compromise

High severity.

### [Cross-Site Scripting (XSS)](level-2/04-xss.md)

**Q53.** What is XSS?

**Answer:** Injecting script into a page so it runs in other users' browsers

Impact depends on context.

**Q54.** What's the main defense?

**Answer:** Context-aware output encoding (plus CSP)

Encode on output.

**Q55.** What are the XSS types?

**Answer:** Reflected, stored and DOM-based

Different delivery paths.

**Q56.** What can XSS steal?

**Answer:** Session tokens or data and act as the user (mitigate with HttpOnly cookies)

HttpOnly blocks JS reads.

### [Broken Authentication &amp; Sessions](level-2/05-broken-authentication.md)

**Q57.** What is credential stuffing?

**Answer:** Trying leaked username/password pairs against other sites

Use MFA and rate limits.

**Q58.** Which session practice is correct?

**Answer:** Regenerate the session ID after login and use Secure/HttpOnly cookies

Prevent fixation.

**Q59.** How do you defend against brute force?

**Answer:** Rate limiting, lockout/backoff and MFA

Detect and slow.

**Q60.** Why give generic login errors?

**Answer:** Avoid revealing whether a username exists

User enumeration.

### [Broken Access Control &amp; IDOR](level-2/06-access-control-idor.md)

**Q61.** What is IDOR?

**Answer:** Accessing another user's object by changing an identifier because authorization isn't checked

Broken access control.

**Q62.** How is IDOR fixed?

**Answer:** Enforce server-side authorization on every object access

Check ownership.

**Q63.** Why aren't random IDs enough?

**Answer:** Unpredictable IDs are not authorization; they can leak

Always check access.

**Q64.** Which OWASP category is IDOR in?

**Answer:** Broken Access Control

#1 in recent lists.

### [SSRF, File Upload &amp; Path Traversal](level-2/07-ssrf-upload-traversal.md)

**Q65.** What is SSRF?

**Answer:** Making the server send requests to internal or attacker-chosen resources

Cloud metadata risk.

**Q66.** What controls prevent SSRF?

**Answer:** Allow-lists for destinations, blocking internal ranges and metadata endpoints

Network egress rules.

**Q67.** What risk comes from file upload?

**Answer:** Uploading executable content; validate type/content and store outside webroot

Rename and scan.

**Q68.** What is path traversal?

**Answer:** Using ../ sequences to read files outside the intended directory

Canonicalize and restrict.

### [Misconfiguration &amp; Vulnerable Components](level-2/08-misconfiguration-components.md)

**Q69.** What is a common misconfiguration?

**Answer:** Default credentials, verbose errors and exposed admin panels

Hardening checklists.

**Q70.** What are vulnerable components?

**Answer:** Outdated libraries with known CVEs

Software composition analysis.

**Q71.** Why track dependencies?

**Answer:** Known vulnerabilities in libraries are an easy way in

SBOM.

**Q72.** What reduces misconfiguration?

**Answer:** Hardened baselines, automated checks and least privilege

Repeatable config.

### [API Security Testing](level-2/09-api-security-testing.md)

**Q73.** What's a key API risk?

**Answer:** Broken object-level authorization (BOLA)

IDOR in APIs.

**Q74.** What should APIs validate?

**Answer:** Authentication, authorization, input schema and rate limits

Defense in depth.

**Q75.** Why is excessive data exposure a risk?

**Answer:** APIs returning more fields than needed leak sensitive data

Return only necessary fields.

**Q76.** How do you test APIs?

**Answer:** With a proxy/Postman against an authorized target, checking auth and object access

Methodical.

### [Project — Assessing OWASP Juice Shop](level-2/10-project-juice-shop.md)

**Q77.** What is OWASP Juice Shop?

**Answer:** A deliberately vulnerable web app for legal practice

Run it locally.

**Q78.** What do you document per challenge?

**Answer:** Steps, evidence, impact and the fix

Report style.

**Q79.** Why run it locally?

**Answer:** Authorization: it's your own target

Legal practice.

**Q80.** What do you learn from the project?

**Answer:** Recognizing and fixing common web vulnerability classes

Map to OWASP Top 10.

## Level 3 · Infrastructure & Network Testing

### [Vulnerability Scanning &amp; Validation](level-3/01-vuln-scanning-validation.md)

**Q81.** Why validate scanner findings?

**Answer:** Scanners produce false positives; confirm before reporting

Evidence.

**Q82.** What's a false positive?

**Answer:** A reported issue that isn't actually exploitable/present

Validate manually.

**Q83.** What does authenticated scanning improve?

**Answer:** Coverage and accuracy by seeing installed software and configuration

Credentialed scans.

**Q84.** How do you prioritize validated findings?

**Answer:** By severity, exploitability, exposure and asset value

Risk-based.

### [Exploitation Fundamentals &amp; Metasploit](level-3/02-exploitation-metasploit.md)

**Q85.** What is Metasploit used for in a lab?

**Answer:** A framework to validate exploitability of known vulnerabilities in authorized tests

Use only on lab/authorized targets.

**Q86.** Why prefer minimal-risk validation?

**Answer:** Avoid disrupting production; prove impact without damage

Safe testing.

**Q87.** What should you record for each exploit?

**Answer:** Exact steps, output and impact for the report

Reproducible.

**Q88.** When is exploitation allowed?

**Answer:** Only within the authorized scope and rules of engagement

RoE.

### [Password Attacks &amp; Cracking](level-3/03-password-attacks.md)

**Q89.** What does password cracking test?

**Answer:** Weak or reused passwords and hashing strength, in an authorized test

Use offline hashes you're authorized to test.

**Q90.** How many combinations are there for an 8-character password using 62 symbols?

```python
print(f'{62**8:,}')
```

**Answer:** 218,340,105,584,896

Length and character set drive search space. Verified by running it.

**Q91.** Why slow salted hashes (bcrypt/Argon2)?

**Answer:** They make offline guessing far more expensive

Defense.

**Q92.** What defense stops credential reuse damage?

**Answer:** Unique passwords with a manager and MFA

Reduce blast radius.

### [Linux Privilege Escalation](level-3/04-linux-privesc.md)

**Q93.** What is Linux privilege escalation?

**Answer:** Gaining higher privileges on a host via misconfigurations or vulnerabilities

Report the weakness.

**Q94.** Which misconfiguration commonly enables privesc?

**Answer:** Overly permissive sudo rules or SUID binaries

Least privilege.

**Q95.** How do defenders reduce risk?

**Answer:** Least privilege, patching, removing unneeded SUID and monitoring

Harden.

**Q96.** What do enumeration scripts (LinPEAS) do?

**Answer:** Automate checks for common misconfigurations in your lab

Learn what they check.

### [Windows Privilege Escalation](level-3/05-windows-privesc.md)

**Q97.** What commonly enables Windows privesc?

**Answer:** Unquoted service paths, weak service permissions and stored credentials

Misconfigurations.

**Q98.** How do defenders mitigate?

**Answer:** Least privilege, patching, secure service configuration and application control

Hardening.

**Q99.** Why does credential hygiene matter?

**Answer:** Stored or reused credentials enable lateral movement

Use LAPS and tiering.

**Q100.** What should you document?

**Answer:** The misconfiguration, evidence and remediation

Fixable findings.

### [Active Directory Attack Paths](level-3/06-active-directory.md)

**Q101.** Why is Active Directory a prime target?

**Answer:** It controls identities and access across the domain

Compromise = widespread access.

**Q102.** Which AD weakness class is common?

**Answer:** Weak service account passwords, excessive privileges and legacy protocols

Kerberoasting awareness.

**Q103.** What helps AD defense?

**Answer:** Tiered administration, strong service account passwords/gMSAs, monitoring and patching

Defense.

**Q104.** Why map attack paths with BloodHound in a lab?

**Answer:** Visualize privilege relationships defenders can break

Use on authorized data.

### [Pivoting &amp; Lateral Movement](level-3/07-pivoting-lateral-movement.md)

**Q105.** What is lateral movement?

**Answer:** Moving from one compromised host to others inside a network

Segmentation limits it.

**Q106.** What is pivoting?

**Answer:** Using a compromised host as a relay to reach other network segments

Authorized lab techniques.

**Q107.** What stops lateral movement?

**Answer:** Network segmentation, least privilege and monitoring

Defense in depth.

**Q108.** How do defenders detect it?

**Answer:** Unusual authentication and east-west traffic monitoring

Detection engineering.

### [Wireless Network Testing](level-3/08-wireless-testing.md)

**Q109.** What do wireless tests cover?

**Answer:** Encryption strength, authentication configuration and rogue access points on authorized networks

Authorization.

**Q110.** Which protocol should replace WEP/WPA?

**Answer:** WPA2/WPA3 with strong passphrases

Modern security.

**Q111.** Why disable WPS?

**Answer:** It has weaknesses that allow PIN attacks

Hardening.

**Q112.** What's an evil twin?

**Answer:** A rogue AP imitating a legitimate network; defend with WPA3/802.1X and user awareness

Detect rogue APs.

### [Social Engineering &amp; Authorized Phishing Simulation](level-3/09-social-engineering.md)

**Q113.** What is social engineering?

**Answer:** Manipulating people into revealing information or taking actions

Targets humans.

**Q114.** When may you run phishing simulations?

**Answer:** With explicit authorization and agreed rules

Ethics and law.

**Q115.** What reduces phishing risk?

**Answer:** Awareness training, MFA and reporting mechanisms

Culture.

**Q116.** What should phishing tests avoid?

**Answer:** Harmful content and collecting real sensitive data

Minimize harm.

### [Project — Internal Network Pentest](level-3/10-project-internal-pentest.md)

**Q117.** What does the internal pentest project combine?

**Answer:** Recon, scanning, validation, privesc and lateral movement in an authorized lab

End to end.

**Q118.** What's the deliverable?

**Answer:** A report with findings, evidence, risk and remediation

Actionable.

**Q119.** How do you ensure safety?

**Answer:** Stay in scope, avoid destructive actions and snapshot the lab

Responsible testing.

**Q120.** How do you validate remediation?

**Answer:** Retest after fixes

Close the loop.

## Level 4 · Professional Practice

### [Post-Exploitation &amp; Evidence Handling](level-4/01-post-exploitation.md)

**Q121.** What is post-exploitation for?

**Answer:** Assessing impact: what data and systems could be reached, within scope

Demonstrate risk.

**Q122.** What must you do with persistence mechanisms?

**Answer:** Avoid or remove them and record what was changed

Clean up.

**Q123.** What data should you handle carefully?

**Answer:** Any sensitive data found: minimize, protect and report

Ethics.

**Q124.** What do you document?

**Answer:** Actions, timestamps and evidence for the report

Traceability.

### [Writing the Penetration Test Report](level-4/02-writing-the-report.md)

**Q125.** What makes a good pentest report?

**Answer:** Executive summary, scoped methodology, prioritized findings with evidence and remediation

Two audiences.

**Q126.** What does the executive summary state?

**Answer:** Overall risk and key recommendations in business terms

For decision makers.

**Q127.** How should findings be rated?

**Answer:** Consistently with CVSS plus business context

Defensible.

**Q128.** What makes remediation advice useful?

**Answer:** Specific, prioritized and verifiable fixes

Actionable.

### [Scoping, Contracts &amp; Rules of Engagement](level-4/03-scoping-roe.md)

**Q129.** What's scoping for?

**Answer:** Defining targets, methods, timing and limits in writing before testing

Legal and safe.

**Q130.** What do Rules of Engagement include?

**Answer:** Allowed techniques, testing windows, contacts and escalation

Operational safety.

**Q131.** What if you find a critical issue mid-test?

**Answer:** Follow the RoE to notify the client promptly

Communication.

**Q132.** Why get written authorization?

**Answer:** It's the legal basis for the test

Verbal isn't enough.

### [Threat Modeling &amp; Attack Surface Mapping](level-4/04-threat-modeling.md)

**Q133.** What is threat modeling?

**Answer:** Identifying assets, threats and mitigations early in design

STRIDE.

**Q134.** What does STRIDE stand for?

**Answer:** Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege

Threat categories.

**Q135.** When is threat modeling most valuable?

**Answer:** Early in design, before code is built

Cheaper fixes.

**Q136.** What does a data flow diagram show?

**Answer:** Trust boundaries and where data crosses them

Focus attention.

### [Cloud Penetration Testing Basics](level-4/05-cloud-pentesting.md)

**Q137.** What differs in cloud pentesting?

**Answer:** Shared responsibility and provider rules of engagement apply

Check provider policy.

**Q138.** What's a top cloud finding?

**Answer:** Overly permissive IAM and publicly exposed storage

Misconfiguration.

**Q139.** Which approach is appropriate?

**Answer:** Use only authorized accounts and test environments

Authorization.

**Q140.** What should you review?

**Answer:** IAM policies, network exposure and logging configuration

Config reviews.

### [Mobile Application Testing Basics](level-4/06-mobile-testing.md)

**Q141.** What do mobile app tests cover?

**Answer:** Insecure storage, weak transport security and exposed APIs on authorized apps

OWASP MASVS.

**Q142.** Why not store secrets in the app?

**Answer:** Apps can be decompiled and secrets extracted

Use backends.

**Q143.** What does certificate pinning protect?

**Answer:** Against man-in-the-middle with fraudulent certificates

Test in lab.

**Q144.** What's a safe lab target?

**Answer:** A deliberately vulnerable app on an emulator

Authorized.

### [Red Team, Pentest &amp; Bug Bounty Compared](level-4/07-red-team-bug-bounty.md)

**Q145.** What's the difference between a red team and a pentest?

**Answer:** A red team emulates a real adversary against detection and response, broader than listing vulnerabilities

Objectives-based.

**Q146.** What does a bug bounty require?

**Answer:** Following the program's scope and rules; never testing outside them

Safe harbor depends on rules.

**Q147.** How do you report a bug?

**Answer:** Clearly, with reproduction steps, impact and without exploiting beyond need

Responsible.

**Q148.** Why read the policy?

**Answer:** It defines scope, allowed methods and legal protections

Scope = safe.

### [Certifications &amp; Career Path](level-4/08-certifications-career.md)

**Q149.** Which makes a good certification path to explore?

**Answer:** Entry to advanced options like Security+, eJPT, PNPT and OSCP according to goals

Choose by role.

**Q150.** What builds a career beyond certs?

**Answer:** Practice labs, CTFs, writeups and real engagements

Demonstrate skill.

**Q151.** Why document your lab work?

**Answer:** Portfolio of writeups shows methodology and communication

Professional proof.

**Q152.** What soft skill matters most?

**Answer:** Clear reporting and communication

Value delivered.

### [Staying Legal: Law, Disclosure &amp; Ethics](level-4/09-law-disclosure-ethics.md)

**Q153.** What law is commonly relevant to unauthorized access?

**Answer:** Computer misuse laws (e.g. the CFAA in the US, Computer Misuse Act in the UK)

Authorization matters.

**Q154.** What is responsible disclosure?

**Answer:** Privately informing the vendor and allowing time to fix before public details

Coordinated.

**Q155.** What's the ethical limit?

**Answer:** Minimize harm, stay in scope and protect any data you encounter

Do no harm.

**Q156.** When in doubt about scope?

**Answer:** Stop and ask for written clarification

Always.

### [Capstone — A Complete Engagement End to End](level-4/10-capstone-full-engagement.md)

**Q157.** What does the capstone engagement combine?

**Answer:** Scoping, recon, testing, reporting and retest on an authorized lab

End to end.

**Q158.** How do you prove professionalism?

**Answer:** Clear scope, safe testing, evidence-based findings and a quality report

Deliver value.

**Q159.** What closes an engagement?

**Answer:** Debrief, report delivery and retest of fixes

Follow-through.

**Q160.** What's the long-term goal?

**Answer:** Help organizations reduce risk responsibly

Ethical practice.

## Mixed questions across all levels

**Q161.** A client's scope lists 10.0.1.0/24 only. You discover 10.0.2.15 is also vulnerable. What do you do?

**Answer:** Stop and request written scope clarification; don't test it

Out-of-scope testing is unauthorized.

**Q162.** What is the correct order of the penetration-test lifecycle?

**Answer:** Scoping, reconnaissance, scanning/enumeration, exploitation, post-exploitation, reporting

Each phase feeds the next.

**Q163.** Which recon technique is passive?

**Answer:** Searching WHOIS, certificate transparency and public repositories

No traffic to the target.

**Q164.** How many total addresses are in a /27 network?

```python
import ipaddress
print(ipaddress.ip_network('10.0.0.0/27').num_addresses)
```

**Answer:** 32

2^(32-27). Verified by running it.

**Q165.** An nmap result shows a TCP port 'closed' vs 'filtered'. What's the difference?

**Answer:** Closed replied with RST; filtered gave no reply (a firewall dropped it)

Interpreting scan results.

**Q166.** A scanner reports a critical CVE on a banner version, but the vendor backported the fix. What do you do?

**Answer:** Validate manually; it may be a false positive

Banner checks are inferences.

**Q167.** What severity band is a CVSS 7.5?

```python
s=7.5
print('Critical' if s>=9 else 'High' if s>=7 else 'Medium' if s>=4 else 'Low')
```

**Answer:** High

7.0-8.9 is High. Verified by running it.

**Q168.** A search field's input is concatenated into SQL. What's the correct fix?

**Answer:** Use parameterized queries (and least-privilege DB accounts)

Keep data separate from code.

**Q169.** A comment is displayed unescaped and runs script for other users. What vulnerability is it and the fix?

**Answer:** Stored XSS; apply context-aware output encoding (plus CSP)

Encode on output.

**Q170.** Changing /invoice/1001 to /invoice/1002 shows another user's invoice. What is it and the fix?

**Answer:** IDOR / broken access control; enforce server-side ownership checks

Authorization on every object.

**Q171.** A 'fetch URL' feature can reach http://169.254.169.254/. What's the risk and defense?

**Answer:** SSRF to cloud metadata; allow-list destinations and block internal ranges

Egress controls.

**Q172.** A pentester finds a database port exposed to the internet. What's the recommendation?

**Answer:** Restrict network access by firewall/VPN and require strong authentication

Reduce exposure.

**Q173.** A scan shows an old library with a known RCE. What process finds this routinely?

**Answer:** Software composition analysis / dependency scanning with an SBOM

Track components.

**Q174.** How many 8-character passwords exist over 26 lowercase letters?

```python
print(f'{26**8:,}')
```

**Answer:** 208,827,064,576

Length and alphabet determine the search space. Verified by running it.

**Q175.** Why should password hashes be bcrypt/Argon2 rather than plain SHA-256?

**Answer:** They are slow and salted, making offline guessing expensive

Work factor.

**Q176.** A low-privilege user can run any command via sudo without a password. What's the finding?

**Answer:** Excessive sudo privileges enabling privilege escalation; apply least privilege

Report and fix.

**Q177.** What controls limit lateral movement after one host is compromised?

**Answer:** Network segmentation, least privilege, tiered admin and east-west monitoring

Contain the blast radius.

**Q178.** What must exist before a phishing simulation?

**Answer:** Written authorization with agreed rules, scope and safeguards

Ethics and law.

**Q179.** What belongs in an executive summary?

**Answer:** Overall risk, key findings in business terms and prioritized recommendations

For decision makers.

## Practical coding questions

### Compute a CVSS 3.1 base score

Implement cvss31_base(vector) following the CVSS 3.1 specification (ISS, scope-aware Impact, Exploitability, and the spec's integer-based round-up), plus severity(). Check it on three vectors, including a scope-changed one.

Starter code:

```
import math

WEIGHTS = {
    "AV": {"N": 0.85, "A": 0.62, "L": 0.55, "P": 0.2},
    "AC": {"L": 0.77, "H": 0.44},
    "UI": {"N": 0.85, "R": 0.62},
    "C": {"H": 0.56, "L": 0.22, "N": 0.0},
    "I": {"H": 0.56, "L": 0.22, "N": 0.0},
    "A": {"H": 0.56, "L": 0.22, "N": 0.0},
}
PR = {"U": {"N": 0.85, "L": 0.62, "H": 0.27}, "C": {"N": 0.85, "L": 0.68, "H": 0.5}}

def roundup(x):
    # CVSS 3.1 round-up to one decimal (avoid floating point issues using integer maths)
    return 0.0

def cvss31_base(vector):
    # parse the vector, compute ISS, Impact, Exploitability and the base score per the CVSS 3.1 spec
    return 0.0

def severity(score):
    # 0 None, <4 Low, <7 Medium, <9 High, else Critical
    return ""

for v in [
    "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
    "CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N",
    "CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N",
]:
    s = cvss31_base(v)
    print(s, severity(s))
```

Expected output:

```text
9.8 Critical
6.1 Medium
5.5 Medium
```

??? success "Show a solution"

    ```
    import math

    WEIGHTS = {
        "AV": {"N": 0.85, "A": 0.62, "L": 0.55, "P": 0.2},
        "AC": {"L": 0.77, "H": 0.44},
        "UI": {"N": 0.85, "R": 0.62},
        "C": {"H": 0.56, "L": 0.22, "N": 0.0},
        "I": {"H": 0.56, "L": 0.22, "N": 0.0},
        "A": {"H": 0.56, "L": 0.22, "N": 0.0},
    }
    PR = {"U": {"N": 0.85, "L": 0.62, "H": 0.27}, "C": {"N": 0.85, "L": 0.68, "H": 0.5}}

    def roundup(x):
        i = round(x * 100000)
        return i / 100000.0 if i % 10000 == 0 else (math.floor(i / 10000) + 1) / 10.0

    def cvss31_base(vector):
        m = dict(part.split(":") for part in vector.split("/")[1:])
        scope_changed = m["S"] == "C"
        iss = 1 - (1 - WEIGHTS["C"][m["C"]]) * (1 - WEIGHTS["I"][m["I"]]) * (1 - WEIGHTS["A"][m["A"]])
        if scope_changed:
            impact = 7.52 * (iss - 0.029) - 3.25 * (iss - 0.02) ** 15
        else:
            impact = 6.42 * iss
        exploit = 8.22 * WEIGHTS["AV"][m["AV"]] * WEIGHTS["AC"][m["AC"]] * PR["C" if scope_changed else "U"][m["PR"]] * WEIGHTS["UI"][m["UI"]]
        if impact <= 0:
            return 0.0
        raw = 1.08 * (impact + exploit) if scope_changed else impact + exploit
        return roundup(min(raw, 10))

    def severity(score):
        return "None" if score == 0 else "Low" if score < 4 else "Medium" if score < 7 else "High" if score < 9 else "Critical"

    for v in [
        "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
        "CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N",
        "CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N",
    ]:
        s = cvss31_base(v)
        print(s, severity(s))
    ```

    The well-known AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H vector scores 9.8 Critical. Scores guide priority, but exposure and asset value still decide real risk. Verified by running the solution.

### Turn a scan into an attack-surface report

Parse nmap grepable-style lines into {host: [(port, service, version)]} keeping only open ports, then print one line per host and a flagged line for each risky service (ftp, telnet, mysql, vnc) with remediation advice. Use only against output from systems you're authorized to scan.

Starter code:

```
import re

SCAN = """\
Host: 10.10.10.5 ()  Ports: 21/open/tcp//ftp//vsftpd 2.3.4/, 22/open/tcp//ssh//OpenSSH 8.9/, 80/open/tcp//http//Apache httpd 2.4.49/, 3306/open/tcp//mysql//MySQL 5.7/
Host: 10.10.10.9 ()  Ports: 443/open/tcp//https//nginx 1.25/, 445/closed/tcp//microsoft-ds///
Host: 10.10.10.20 ()  Ports: 23/open/tcp//telnet///, 5900/open/tcp//vnc//VNC (protocol 3.8)/
"""

RISKY = {
    "ftp": "cleartext protocol: prefer SFTP and disable anonymous access",
    "telnet": "cleartext remote shell: replace with SSH",
    "mysql": "database exposed to the network: restrict by firewall",
    "vnc": "remote desktop exposed: tunnel via VPN / require strong auth",
}

def parse(scan):
    # parse nmap grepable lines into {host: [(port, service, version), ...]} keeping only OPEN ports
    return {}

def report(hosts):
    # one line per host "<ip>: N open", then for each risky service (see RISKY) an indented
    # "  [!] <port>/<service> <version> -> <advice>" line
    return []

for line in report(parse(SCAN)):
    print(line)
```

Expected output:

```text
10.10.10.5: 4 open
  [!] 21/ftp vsftpd 2.3.4 -> cleartext protocol: prefer SFTP and disable anonymous access
  [!] 3306/mysql MySQL 5.7 -> database exposed to the network: restrict by firewall
10.10.10.9: 1 open
10.10.10.20: 2 open
  [!] 23/telnet -> cleartext remote shell: replace with SSH
  [!] 5900/vnc VNC (protocol 3.8) -> remote desktop exposed: tunnel via VPN / require strong auth
```

??? success "Show a solution"

    ```
    import re

    SCAN = """\
    Host: 10.10.10.5 ()  Ports: 21/open/tcp//ftp//vsftpd 2.3.4/, 22/open/tcp//ssh//OpenSSH 8.9/, 80/open/tcp//http//Apache httpd 2.4.49/, 3306/open/tcp//mysql//MySQL 5.7/
    Host: 10.10.10.9 ()  Ports: 443/open/tcp//https//nginx 1.25/, 445/closed/tcp//microsoft-ds///
    Host: 10.10.10.20 ()  Ports: 23/open/tcp//telnet///, 5900/open/tcp//vnc//VNC (protocol 3.8)/
    """

    RISKY = {
        "ftp": "cleartext protocol: prefer SFTP and disable anonymous access",
        "telnet": "cleartext remote shell: replace with SSH",
        "mysql": "database exposed to the network: restrict by firewall",
        "vnc": "remote desktop exposed: tunnel via VPN / require strong auth",
    }

    def parse(scan):
        hosts = {}
        for line in scan.splitlines():
            m = re.match(r"Host: (\S+) .*?Ports: (.*)", line)
            if not m:
                continue
            ports = []
            for item in m.group(2).split(", "):
                f = item.strip().strip("/").split("/")
                if f[1] == "open":
                    ports.append((int(f[0]), f[4], f[6] if len(f) > 6 else ""))
            hosts[m.group(1)] = ports
        return hosts

    def report(hosts):
        out = []
        for h, ports in hosts.items():
            out.append(f"{h}: {len(ports)} open")
            for port, svc, ver in ports:
                if svc in RISKY:
                    out.append(f"  [!] {port}/{svc} {ver}".rstrip() + f" -> {RISKY[svc]}")
        return out

    for line in report(parse(SCAN)):
        print(line)
    ```

    Reducing raw scan output to exposed risky services with concrete fixes is exactly what a discovery report does. The parser works on text you already have; it doesn't scan anything. Verified by running the solution.

---

More free practice: the [Ethical Hacking Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
