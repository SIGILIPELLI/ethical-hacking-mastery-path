---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Ethical Hacking Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "A client's scope lists 10.0.1.0/24 only. You discover 10.0.2.15 is also vulnerable. What do you do?",
    "options": [
      "Ignore it",
      "Test it",
      "Publish it",
      "Stop and request written scope clarification; don't test it"
    ],
    "correctIndex": 3,
    "explanation": "Out-of-scope testing is unauthorized."
  },
  {
    "question": "What is the correct order of the penetration-test lifecycle?",
    "options": [
      "Report first",
      "Scoping, reconnaissance, scanning/enumeration, exploitation, post-exploitation, reporting",
      "Exploit first",
      "Recon, report, scope"
    ],
    "correctIndex": 1,
    "explanation": "Each phase feeds the next."
  },
  {
    "question": "Which recon technique is passive?",
    "options": [
      "Searching WHOIS, certificate transparency and public repositories",
      "Directory brute force",
      "nmap SYN scan",
      "Login attempts"
    ],
    "correctIndex": 0,
    "explanation": "No traffic to the target."
  },
  {
    "question": "How many total addresses are in a /27 network?\n```python\nimport ipaddress\nprint(ipaddress.ip_network('10.0.0.0/27').num_addresses)\n```\n",
    "options": [
      "27",
      "30",
      "64",
      "32"
    ],
    "correctIndex": 3,
    "explanation": "2^(32-27). Verified by running it."
  },
  {
    "question": "An nmap result shows a TCP port 'closed' vs 'filtered'. What's the difference?",
    "options": [
      "Closed is open",
      "Closed replied with RST; filtered gave no reply (a firewall dropped it)",
      "Same",
      "Filtered replied"
    ],
    "correctIndex": 1,
    "explanation": "Interpreting scan results."
  },
  {
    "question": "A scanner reports a critical CVE on a banner version, but the vendor backported the fix. What do you do?",
    "options": [
      "Report as is",
      "Ignore",
      "Exploit",
      "Validate manually; it may be a false positive"
    ],
    "correctIndex": 3,
    "explanation": "Banner checks are inferences."
  },
  {
    "question": "What severity band is a CVSS 7.5?\n```python\ns=7.5\nprint('Critical' if s>=9 else 'High' if s>=7 else 'Medium' if s>=4 else 'Low')\n```\n",
    "options": [
      "High",
      "Low",
      "Critical",
      "Medium"
    ],
    "correctIndex": 0,
    "explanation": "7.0-8.9 is High. Verified by running it."
  },
  {
    "question": "A search field's input is concatenated into SQL. What's the correct fix?",
    "options": [
      "Hide errors",
      "Use POST",
      "Use parameterized queries (and least-privilege DB accounts)",
      "Blacklist words"
    ],
    "correctIndex": 2,
    "explanation": "Keep data separate from code."
  },
  {
    "question": "A comment is displayed unescaped and runs script for other users. What vulnerability is it and the fix?",
    "options": [
      "SQLi; parameters",
      "SSRF; allow-list",
      "Stored XSS; apply context-aware output encoding (plus CSP)",
      "CSRF; tokens"
    ],
    "correctIndex": 2,
    "explanation": "Encode on output."
  },
  {
    "question": "Changing /invoice/1001 to /invoice/1002 shows another user's invoice. What is it and the fix?",
    "options": [
      "IDOR / broken access control; enforce server-side ownership checks",
      "SQLi",
      "XSS",
      "CSRF"
    ],
    "correctIndex": 0,
    "explanation": "Authorization on every object."
  },
  {
    "question": "A 'fetch URL' feature can reach http://169.254.169.254/. What's the risk and defense?",
    "options": [
      "DoS",
      "SQLi",
      "XSS",
      "SSRF to cloud metadata; allow-list destinations and block internal ranges"
    ],
    "correctIndex": 3,
    "explanation": "Egress controls."
  },
  {
    "question": "A pentester finds a database port exposed to the internet. What's the recommendation?",
    "options": [
      "Ignore",
      "Restrict network access by firewall/VPN and require strong authentication",
      "Hide banner",
      "Rename port"
    ],
    "correctIndex": 1,
    "explanation": "Reduce exposure."
  },
  {
    "question": "A scan shows an old library with a known RCE. What process finds this routinely?",
    "options": [
      "Phishing",
      "Manual only",
      "Software composition analysis / dependency scanning with an SBOM",
      "Pentest only"
    ],
    "correctIndex": 2,
    "explanation": "Track components."
  },
  {
    "question": "How many 8-character passwords exist over 26 lowercase letters?\n```python\nprint(f'{26**8:,}')\n```\n",
    "options": [
      "26,000",
      "208,827,064,576",
      "1,000,000,000",
      "8,000,000"
    ],
    "correctIndex": 1,
    "explanation": "Length and alphabet determine the search space. Verified by running it."
  },
  {
    "question": "Why should password hashes be bcrypt/Argon2 rather than plain SHA-256?",
    "options": [
      "Law",
      "They're shorter",
      "They are slow and salted, making offline guessing expensive",
      "They're reversible"
    ],
    "correctIndex": 2,
    "explanation": "Work factor."
  },
  {
    "question": "A low-privilege user can run any command via sudo without a password. What's the finding?",
    "options": [
      "Excessive sudo privileges enabling privilege escalation; apply least privilege",
      "Normal",
      "Nothing",
      "Strong"
    ],
    "correctIndex": 0,
    "explanation": "Report and fix."
  },
  {
    "question": "What controls limit lateral movement after one host is compromised?",
    "options": [
      "Network segmentation, least privilege, tiered admin and east-west monitoring",
      "Shared admin",
      "More users",
      "Flat network"
    ],
    "correctIndex": 0,
    "explanation": "Contain the blast radius."
  },
  {
    "question": "What must exist before a phishing simulation?",
    "options": [
      "Verbal OK",
      "A tool",
      "Written authorization with agreed rules, scope and safeguards",
      "Nothing"
    ],
    "correctIndex": 2,
    "explanation": "Ethics and law."
  },
  {
    "question": "What belongs in an executive summary?",
    "options": [
      "Commands",
      "Hashes",
      "Raw logs",
      "Overall risk, key findings in business terms and prioritized recommendations"
    ],
    "correctIndex": 3,
    "explanation": "For decision makers."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Compute a CVSS 3.1 base score

Implement cvss31_base(vector) following the CVSS 3.1 specification (ISS, scope-aware Impact, Exploitability, and the spec's integer-based round-up), plus severity(). Check it on three vectors, including a scope-changed one.

**Starter code**

```python
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

**Expected output**

```text
9.8 Critical
6.1 Medium
5.5 Medium
```

<details><summary>Show solution and explanation</summary>

<pre><code>import math

WEIGHTS = {
    &quot;AV&quot;: {&quot;N&quot;: 0.85, &quot;A&quot;: 0.62, &quot;L&quot;: 0.55, &quot;P&quot;: 0.2},
    &quot;AC&quot;: {&quot;L&quot;: 0.77, &quot;H&quot;: 0.44},
    &quot;UI&quot;: {&quot;N&quot;: 0.85, &quot;R&quot;: 0.62},
    &quot;C&quot;: {&quot;H&quot;: 0.56, &quot;L&quot;: 0.22, &quot;N&quot;: 0.0},
    &quot;I&quot;: {&quot;H&quot;: 0.56, &quot;L&quot;: 0.22, &quot;N&quot;: 0.0},
    &quot;A&quot;: {&quot;H&quot;: 0.56, &quot;L&quot;: 0.22, &quot;N&quot;: 0.0},
}
PR = {&quot;U&quot;: {&quot;N&quot;: 0.85, &quot;L&quot;: 0.62, &quot;H&quot;: 0.27}, &quot;C&quot;: {&quot;N&quot;: 0.85, &quot;L&quot;: 0.68, &quot;H&quot;: 0.5}}

def roundup(x):
    i = round(x * 100000)
    return i / 100000.0 if i % 10000 == 0 else (math.floor(i / 10000) + 1) / 10.0

def cvss31_base(vector):
    m = dict(part.split(&quot;:&quot;) for part in vector.split(&quot;/&quot;)[1:])
    scope_changed = m[&quot;S&quot;] == &quot;C&quot;
    iss = 1 - (1 - WEIGHTS[&quot;C&quot;][m[&quot;C&quot;]]) * (1 - WEIGHTS[&quot;I&quot;][m[&quot;I&quot;]]) * (1 - WEIGHTS[&quot;A&quot;][m[&quot;A&quot;]])
    if scope_changed:
        impact = 7.52 * (iss - 0.029) - 3.25 * (iss - 0.02) ** 15
    else:
        impact = 6.42 * iss
    exploit = 8.22 * WEIGHTS[&quot;AV&quot;][m[&quot;AV&quot;]] * WEIGHTS[&quot;AC&quot;][m[&quot;AC&quot;]] * PR[&quot;C&quot; if scope_changed else &quot;U&quot;][m[&quot;PR&quot;]] * WEIGHTS[&quot;UI&quot;][m[&quot;UI&quot;]]
    if impact &lt;= 0:
        return 0.0
    raw = 1.08 * (impact + exploit) if scope_changed else impact + exploit
    return roundup(min(raw, 10))

def severity(score):
    return &quot;None&quot; if score == 0 else &quot;Low&quot; if score &lt; 4 else &quot;Medium&quot; if score &lt; 7 else &quot;High&quot; if score &lt; 9 else &quot;Critical&quot;

for v in [
    &quot;CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H&quot;,
    &quot;CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N&quot;,
    &quot;CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N&quot;,
]:
    s = cvss31_base(v)
    print(s, severity(s))
</code></pre>

<p>The well-known AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H vector scores 9.8 Critical. Scores guide priority, but exposure and asset value still decide real risk. Verified by running the solution.</p>
</details>

### Turn a scan into an attack-surface report

Parse nmap grepable-style lines into {host: [(port, service, version)]} keeping only open ports, then print one line per host and a flagged line for each risky service (ftp, telnet, mysql, vnc) with remediation advice. Use only against output from systems you're authorized to scan.

**Starter code**

```python
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

**Expected output**

```text
10.10.10.5: 4 open
  [!] 21/ftp vsftpd 2.3.4 -> cleartext protocol: prefer SFTP and disable anonymous access
  [!] 3306/mysql MySQL 5.7 -> database exposed to the network: restrict by firewall
10.10.10.9: 1 open
10.10.10.20: 2 open
  [!] 23/telnet -> cleartext remote shell: replace with SSH
  [!] 5900/vnc VNC (protocol 3.8) -> remote desktop exposed: tunnel via VPN / require strong auth
```

<details><summary>Show solution and explanation</summary>

<pre><code>import re

SCAN = &quot;&quot;&quot;\
Host: 10.10.10.5 ()  Ports: 21/open/tcp//ftp//vsftpd 2.3.4/, 22/open/tcp//ssh//OpenSSH 8.9/, 80/open/tcp//http//Apache httpd 2.4.49/, 3306/open/tcp//mysql//MySQL 5.7/
Host: 10.10.10.9 ()  Ports: 443/open/tcp//https//nginx 1.25/, 445/closed/tcp//microsoft-ds///
Host: 10.10.10.20 ()  Ports: 23/open/tcp//telnet///, 5900/open/tcp//vnc//VNC (protocol 3.8)/
&quot;&quot;&quot;

RISKY = {
    &quot;ftp&quot;: &quot;cleartext protocol: prefer SFTP and disable anonymous access&quot;,
    &quot;telnet&quot;: &quot;cleartext remote shell: replace with SSH&quot;,
    &quot;mysql&quot;: &quot;database exposed to the network: restrict by firewall&quot;,
    &quot;vnc&quot;: &quot;remote desktop exposed: tunnel via VPN / require strong auth&quot;,
}

def parse(scan):
    hosts = {}
    for line in scan.splitlines():
        m = re.match(r&quot;Host: (\S+) .*?Ports: (.*)&quot;, line)
        if not m:
            continue
        ports = []
        for item in m.group(2).split(&quot;, &quot;):
            f = item.strip().strip(&quot;/&quot;).split(&quot;/&quot;)
            if f[1] == &quot;open&quot;:
                ports.append((int(f[0]), f[4], f[6] if len(f) &gt; 6 else &quot;&quot;))
        hosts[m.group(1)] = ports
    return hosts

def report(hosts):
    out = []
    for h, ports in hosts.items():
        out.append(f&quot;{h}: {len(ports)} open&quot;)
        for port, svc, ver in ports:
            if svc in RISKY:
                out.append(f&quot;  [!] {port}/{svc} {ver}&quot;.rstrip() + f&quot; -&gt; {RISKY[svc]}&quot;)
    return out

for line in report(parse(SCAN)):
    print(line)
</code></pre>

<p>Reducing raw scan output to exposed risky services with concrete fixes is exactly what a discovery report does. The parser works on text you already have; it doesn&#x27;t scan anything. Verified by running the solution.</p>
</details>
