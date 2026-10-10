---
description: "Ethical Hacking Mastery Path Level 3: lab infrastructure testing — scanning, exploitation, privilege escalation, Active Directory, pivoting, wireless."
---

# Level 3 · Infrastructure & Network Testing

Level 2 stayed inside one web app. Level 3 takes on hosts and networks: scanning them for
weaknesses, gaining an initial foothold through a known-vulnerable service, escalating from a
limited account to full control, moving through a network, and understanding how Windows domains
fall. Everything here runs against machines you build in the lab from Level 1 — intentionally
vulnerable VMs such as Metasploitable and retired practice boxes.

The emphasis is on **understanding why** each step works, not on memorising commands. A foothold is
a known CVE met with a public module; privilege escalation is a misconfiguration; lateral movement
is reused credentials. Once you see the pattern, the specific tool matters less.

## Modules

1. **[Vulnerability Scanning & Validation](01-vuln-scanning-validation.md)** — automated scanners,
   their false positives, and manual validation.
2. **[Exploitation Fundamentals & Metasploit](02-exploitation-metasploit.md)** — what an exploit
   is, payloads and shells, driven through the Metasploit Framework in the lab.
3. **[Password Attacks & Cracking](03-password-attacks.md)** — hashing, wordlists, and cracking
   hashes **you generate yourself** with Hashcat and John.
4. **[Linux Privilege Escalation](04-linux-privesc.md)** — SUID binaries, sudo rules, cron and
   capabilities.
5. **[Windows Privilege Escalation](05-windows-privesc.md)** — services, tokens, and unquoted
   paths.
6. **[Active Directory Attack Paths](06-active-directory.md)** — how a domain is enumerated and why
   it falls: Kerberos, hashes and trust.
7. **[Pivoting & Lateral Movement](07-pivoting-lateral-movement.md)** — using one foothold to reach
   a network you otherwise couldn't.
8. **[Wireless Network Testing](08-wireless-testing.md)** — Wi-Fi security, WPA2/WPA3, and testing
   **your own** access point.
9. **[Social Engineering & Authorized Phishing Simulation](09-social-engineering.md)** — the human
   layer, and running a phishing simulation inside an authorized scope.
10. **[Project — Internal Network Pentest](10-project-internal-pentest.md)** — foothold to domain,
    documented at every step.
