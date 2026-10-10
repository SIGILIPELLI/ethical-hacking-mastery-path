---
description: "Active scanning with Nmap: host discovery, port states, SYN and connect scans, version and OS detection, timing and NSE — against your own lab."
---

# Active Scanning with Nmap

Recon told you which hosts probably exist. Scanning confirms which are up and which ports they
have open. **Nmap** is the standard tool, and it does far more than "list open ports" — host
discovery, version detection, OS fingerprinting and a whole scripting engine. This lesson is
hands-on; every command is one you run against **your own lab** (or your own machine).

!!! danger "In-scope targets only"
    Nmap sends packets *to the target*. That is active, detectable, and — against a system you
    don't own or have permission to test — potentially a crime. Everything below runs against your
    lab VMs or `127.0.0.1` (your own machine). Nothing else.

## Host discovery

Before scanning ports, find which hosts are alive. On a local network Nmap uses ARP (reliable on
the same subnet); across networks it uses ICMP and TCP probes.

```bash
# Ping-sweep a range to find live hosts (no port scan)
nmap -sn 192.168.56.0/24
```

`-sn` means "no port scan — just discovery." You get back a list of hosts that responded.

## Port states

Nmap reports each scanned port as one of:

| State | Meaning |
|-------|---------|
| **open** | A service is listening and accepting connections |
| **closed** | The host answered, but nothing is listening there |
| **filtered** | No useful answer — a firewall likely dropped the probe |
| **open\|filtered** | Can't tell open from filtered (common on UDP) |

## Scan types

- **`-sS` SYN scan** (default when run as root) — sends a SYN, reads the reply, sends RST instead
  of completing the handshake. Fast; needs raw-socket privileges.
- **`-sT` TCP connect scan** — completes the full handshake using the OS. Used when you lack
  raw-socket privileges; more detectable and a little slower.
- **`-sU` UDP scan** — for UDP services; slow and ambiguous (see the networking lesson).
- **`-sV` version detection** — probes open ports to identify the service *and its version*.
- **`-O` OS detection** — guesses the operating system from TCP/IP stack behaviour.
- **`-p`** — choose ports: `-p 22,80,443`, a range `-p 1-1000`, or everything `-p-` (all 65535).

## A real scan (your own machine)

Here is a genuine `-sV` scan run against `127.0.0.1` on the machine this course was written on. It
is included to show you *exactly* what real output looks like — states, the service column, and
Nmap's honesty when it can't identify something:

```console
$ nmap -sV -p 22,80,443,631,5000,7000 127.0.0.1
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-10 16:19 +0530
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000090s latency).

PORT     STATE  SERVICE VERSION
22/tcp   closed ssh
80/tcp   closed http
443/tcp  closed https
631/tcp  closed ipp
5000/tcp open   rtsp
7000/tcp open   rtsp
2 services unrecognized despite returning data. ...
```

Read it carefully. Ports 22/80/443/631 are **closed** — the host answered, nothing listening.
Ports 5000 and 7000 are **open**, and `-sV` grabbed data but couldn't fully classify them (the
version-detection fingerprint showed a `Server: AirTunes/...` banner — macOS AirPlay). That last
part is important: **Nmap tells you when it is unsure** rather than guessing. A tester takes
"unrecognized despite returning data" as a cue to go look manually (the enumeration lesson).

## A typical lab target

Against a deliberately vulnerable Linux VM such as Metasploitable, an `nmap -sV` scan returns a
long list of open services — FTP, SSH, Telnet, SMTP, HTTP, SMB, MySQL and more — each with a
version string. Those version strings are the gold: "vsftpd 2.3.4" or "an old Samba" points you
straight at known vulnerabilities (the CVE/CVSS lesson). The exact versions depend on which image
and release you run, so build the VM and read *your* scan rather than trusting any number printed
here — this course does not invent version strings for a box it did not scan.

## Timing and output

```bash
nmap -sV -T4 -p- 192.168.56.101           # -T4 = faster timing (0 slowest … 5 fastest)
nmap -sV 192.168.56.101 -oN scan.txt      # save normal output to a file
nmap -sV 192.168.56.101 -oA host1         # save in all formats (.nmap, .gnmap, .xml)
```

Always save output (`-oN`/`-oA`). The report is built from it, and re-running a long scan to
recover output you didn't save is wasted time. Higher `-T` is faster but noisier and can miss
results on congested links; `-T4` is a common default for a responsive lab.

## The Nmap Scripting Engine (NSE)

`-sC` runs a set of default scripts that probe deeper — grabbing banners, enumerating SMB shares,
checking for well-known issues. `--script` runs specific ones.

```bash
nmap -sC -sV 192.168.56.101               # default scripts + versions (very common combo)
nmap --script http-title,http-headers -p 80 192.168.56.101
nmap --script vuln 192.168.56.101         # run vuln-detection scripts (noisy)
```

`nmap -sC -sV -oA host1 <target>` is the workhorse first scan of most engagements: it discovers
services, versions them, and runs the safe default scripts, saving everything.

## How It Actually Works

Why can a SYN scan (`-sS`) be faster and lighter than a connect scan (`-sT`) for the same result?
It comes down to *where* the work happens. A connect scan asks the operating system to make a full
connection via the normal `connect()` system call: the OS completes the three-way handshake
(SYN → SYN-ACK → ACK), hands your program a connected socket, and *then* you tear it down. The
listening application is notified of a real connection, and the OS does all the bookkeeping of an
established session.

A SYN scan bypasses the OS's connection machinery by crafting raw packets itself: it sends the SYN
directly, reads the raw SYN-ACK or RST reply, and sends a RST to abandon the half-open connection
before it ever completes. Nmap, not the OS, tracks which probes are outstanding, so it can have
thousands in flight at once and interpret replies in bulk. That is why it needs raw-socket
privileges (root) — it is speaking TCP by hand — and why it is faster: no per-port OS connection
setup, no application notification, just "SYN out, classify the reply." The result is identical
(SYN-ACK = open, RST = closed, silence = filtered) because, as the networking lesson showed, it is
the target's *kernel* that decides the reply based on whether a service has reserved the port.

Version detection (`-sV`) works differently again: once a port is known open, Nmap connects and
either reads the banner the service volunteers or sends a battery of probes and matches the
responses against a large database of known service fingerprints. When nothing matches — as with
the AirTunes service above — it prints the raw fingerprint and admits it doesn't know, because a
false version claim would be worse than an honest "unrecognized."

## Common mistakes and pitfalls

- **Assuming the service from the port number.** Port 5000 above wasn't a dev web server; it was
  AirPlay. Always confirm with `-sV`, never guess from the port.
- **Scanning only the top 1000 ports and declaring the host mapped.** Services hide on high ports.
  Do a `-p-` sweep at least once per host.
- **Running `-T5` against a fragile or remote target.** Too aggressive; you miss ports or disrupt
  the service. `-T4` is usually the sweet spot; slow down for unreliable links.
- **Not saving output.** Use `-oA`. Every time.
- **Trusting `--script vuln` blindly.** It flags *possible* issues from banners; it does not prove
  exploitability. Validate manually (Level 3 lesson 1).
- **Forgetting UDP entirely.** Important services (DNS, SNMP) are UDP-only. A TCP-only scan misses
  them.

## Exercise

1. Run `nmap -sV -p 1-1000 127.0.0.1` on your own machine and save it with `-oN localscan.txt`.
   Record which ports are open, closed and filtered, and whether `-sV` identified each service.
2. On your lab network, run `nmap -sn <your-host-only-CIDR>` to discover live hosts. List them.
3. Pick one live target and run `nmap -sC -sV -oA target1 <ip>`. From the output, write down every
   open port, its service, and its version string.
4. Explain, in your own words, the difference between `-sS` and `-sT`, including why `-sS` needs
   root and why it is faster.
5. Find one port that `-sV` could not fully identify (or set up a service that confuses it). Explain
   what you would do next to identify it manually.
