---
description: "Vulnerability scanning and manual validation: how automated scanners work, their false positives and negatives, and how a tester confirms a real finding."
---

# Vulnerability Scanning & Validation

Level 1 ended with candidate vulnerabilities inferred from versions. Level 3 begins by scaling that
up with **automated vulnerability scanners** — and, crucially, by learning not to trust them. A
scanner produces a long list of *possible* issues; a tester's job is to separate the real,
exploitable findings from the false positives. The discipline of **validation** is what distinguishes
a penetration test from a scan, and it is why "I ran a scanner" is not a penetration test. Everything
here runs against your own lab targets.

## What a vulnerability scanner does

Tools like **Nessus**, **OpenVAS/Greenbone**, **Qualys** and Nmap's `--script vuln` work roughly
the same way:

1. **Discover** hosts and open ports (as in Level 1).
2. **Identify** services and versions (banners, probes, fingerprints).
3. **Match** those versions and configurations against a database of known vulnerability
   signatures.
4. **Report** each match with a severity (usually CVSS) and a description.

Some scanners also do light **safe checks** (e.g. confirming a default credential works) and have an
optional, riskier "exploit/paranoid" mode. Most of the output, though, is *inference from version*:
"you run version X; version X has CVE-Y; therefore you may be vulnerable to Y."

!!! note "Scanner availability"
    Nessus (Tenable) is commercial with a limited free "Essentials" tier; OpenVAS/Greenbone is open
    source and installable in your lab. This course does not ship scanner output it didn't generate;
    install OpenVAS or use Nmap's `--script vuln` against your lab and read *your* results. The
    method below applies to whichever scanner you run.

```bash
# A lightweight, free "scanner" you can run now, against your lab:
nmap -sV --script vuln <lab-host> -oN vulnscan.txt
```

## Why scanners are not enough

- **False positives.** The scanner saw version X and flagged CVE-Y, but the vulnerable feature is
  disabled, the version is back-patched (common on enterprise Linux — the banner says "1.0" but the
  vendor patched the bug while keeping the version string), or the configuration isn't the
  vulnerable one. The finding is listed; it isn't real.
- **False negatives.** Scanners miss what isn't in their signature database: business-logic flaws,
  most access-control bugs (IDOR), chained vulnerabilities, and anything novel. A clean scan does
  **not** mean a secure system.
- **No impact context.** A scanner rates a bug by generic CVSS; it doesn't know this host holds the
  crown-jewel database or sits on an isolated segment. Severity in the real environment can be very
  different.
- **No chaining.** The real impact often comes from combining a medium-severity info leak with a
  medium misconfiguration to get a critical outcome — something no per-finding scanner reports.

## Validation: turning a candidate into a finding

For each scanner hit worth pursuing:

1. **Confirm the version/config is really present.** Re-check the banner, and where possible the
   actual behaviour, not just the reported version. Beware back-ports.
2. **Confirm the vulnerable code path is reachable.** Is the affected feature enabled and exposed to
   you? A vuln in a disabled module isn't exploitable.
3. **Prove it safely.** Use the least-intrusive proof that establishes the bug: a version-specific
   behaviour, a safe check, or (where the RoE allows and the target is in your lab) a controlled
   exploitation that stops at proof-of-access. Don't run a destructive exploit to "confirm" a bug.
4. **Assess true severity.** Adjust CVSS for the environment (Level 1 lesson 9): reachability,
   data sensitivity, compensating controls.
5. **Record evidence.** The command, the response, the proof — everything the report and a retest
   will need.

Validation is where `searchsploit` and the exploit lesson (next) come in: if a scanner flags a CVE
and a public exploit exists, validating often means running that exploit *in the lab* to confirm it
fires against *this* target.

## A worked example (the reasoning)

A scan of a lab host flags three issues:

| Scanner says | Validation | Verdict |
|--------------|-----------|---------|
| "Service X vX.Y — CVE-A, Critical" | Banner confirms vX.Y; feature enabled; lab exploit from Exploit-DB gets a shell | **Real — Critical.** Document with evidence. |
| "TLS supports an old protocol — Medium" | `testssl.sh` confirms the old protocol is offered | **Real — Medium.** Config finding. |
| "Service Z — CVE-C, High" | Banner says vulnerable version, but the vulnerable module is not loaded; exploit fails | **False positive.** Note it as tested-and-not-exploitable. |

The report contains the two real findings with evidence and the one explicitly-dismissed false
positive — telling the client you *checked* it, which is part of demonstrating thoroughness.

## How It Actually Works

Why do scanners produce so many false positives and negatives when they're built by experts? Because
a scanner is fundamentally a **pattern-matcher over observable signals**, and the signals it can
cheaply observe (version strings, open ports, response fingerprints) are a lossy proxy for the thing
that actually matters (is this specific code path, in this specific configuration, reachable and
exploitable *right now*). The scanner optimises for breadth and speed across thousands of hosts, so
it must infer from surface signals rather than prove by exploitation — proving would be slow,
risky, and sometimes destructive. Inference from a version is a reasonable *bet*, but it's a bet:
back-ported patches break the "version ⇒ vulnerable" link (the signal is intact but the bug is
gone), and disabled features break the "vulnerable ⇒ reachable" link. Hence false positives.

False negatives come from the opposite limit: a signature database can only match *known* patterns,
and the highest-value findings in real tests — broken access control, business-logic abuse, chained
exploits — have no universal signature because they're specific to one application's logic. There's
no banner for "this endpoint forgot to check ownership." A scanner literally cannot see them. This
is the structural reason a penetration test is more than a scan: the human supplies the two things
the scanner can't — *validation* (closing the gap between "matches a signature" and "actually
exploitable") and *judgement* (finding and chaining the logic flaws no database lists, and rating
impact in the real environment). The scanner is a force-multiplier for the mechanical first pass;
the value a tester adds is precisely the part that can't be reduced to pattern-matching.

## Common mistakes and pitfalls

- **Submitting scanner output as a pentest report.** It isn't one. Validate, chain, contextualise,
  and add the findings scanners can't see.
- **Trusting "Critical" without validating.** Back-ports and disabled features turn Criticals into
  false positives constantly.
- **Trusting a clean scan as "secure."** False negatives abound; the absence of scanner hits is not
  the absence of vulnerabilities.
- **Running aggressive/exploit-mode scans against fragile or out-of-scope systems.** They can crash
  services. Keep intrusive scanning to the lab and within the RoE.
- **Ignoring environment in severity.** Report the *contextual* severity, not the raw CVSS, and say
  why.
- **Not recording dismissed false positives.** "Tested, not exploitable" is valuable evidence of
  coverage.

## Exercise

1. Run `nmap -sV --script vuln` (or install OpenVAS) against a lab target. Save the output. How many
   candidate issues did it report?
2. Pick three candidates and validate each: confirm the version/config, check reachability, and
   decide real vs. false positive. Record your reasoning for each.
3. For one validated-real finding, adjust its CVSS for your lab environment and justify any change
   from the scanner's default rating.
4. Identify one *class* of vulnerability from Level 2 (e.g. IDOR) that a version-based scanner would
   not find, and explain why in terms of signatures.
5. In your own words, explain why "version ⇒ vulnerable" is a bet and name the two things that break
   the link (producing false positives).
