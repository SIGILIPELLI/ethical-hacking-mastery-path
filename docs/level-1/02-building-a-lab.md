---
description: "Build a legal, isolated practice lab: a hypervisor, host-only networking, and deliberately vulnerable targets kept off the internet."
---

# Building a Legal, Isolated Practice Lab

The previous lesson said every technique in this course is practised against systems you own. This
lesson is how you build those systems. A home lab is a few virtual machines on your own computer,
wired together on a private network that cannot reach the internet or your real home network. Inside
that bubble you can scan, exploit and break things freely — because you own every machine and
nothing you do leaves the bubble.

## What you need

- **A hypervisor** — software that runs virtual machines. Free options: **VirtualBox** (Windows,
  Linux, Intel macOS), **VMware Workstation Player / Fusion**, or the Linux-native **KVM/QEMU**. On
  Apple-silicon Macs, UTM (QEMU front-end) runs ARM VMs; some x86 security VMs run only slowly or
  not at all under emulation, which is worth knowing before you plan around them.
- **An attacker VM** — **Kali Linux** or **Parrot OS**. Both ship with the tooling this course
  uses (Nmap, Metasploit, Burp, Hashcat, John, Wireshark) preinstalled.
- **Target VMs** — machines built to be vulnerable, so you have something legal to attack:
    - **Metasploitable 2 / 3** — a deliberately broken Linux server.
    - **OWASP Juice Shop** and **DVWA** — deliberately vulnerable web apps (used in Level 2).
    - **VulnHub** images and retired practice boxes — downloadable vulnerable VMs.
- **Disk and RAM** — budget ~8 GB RAM for two VMs running at once and 60–80 GB of disk. Less works;
  it is slower.

## The one rule that keeps it legal: isolation

The lab must not be able to reach anything you do not own. The mechanism is the **virtual network
mode** you assign each VM's adapter. In VirtualBox and VMware the relevant modes are:

| Mode | VM can reach… | Use for the lab? |
|------|---------------|------------------|
| **NAT** | the internet, via the host | Only briefly, to install updates |
| **Bridged** | your real LAN and its devices | **No** — this puts the VM on your home network |
| **Host-only** | the host and other host-only VMs | **Yes** — isolated from the internet and LAN |
| **Internal** | only other VMs on the same internal net | **Yes** — even more isolated (no host) |

The attacker and targets all go on a **host-only** (or internal) network. That way the attacker VM
can reach the targets, the targets can reach each other, and *nothing* can reach the internet or
the other devices in your home. This matters for two reasons: a deliberately vulnerable VM is a
genuine risk if it is reachable from the internet (someone else could compromise it), and it
guarantees that a misfired scan or exploit cannot stray onto a system you do not own.

!!! warning "Never bridge a vulnerable VM"
    A Metasploitable box on bridged networking is a wide-open server on your home LAN that your
    neighbours' malware, or anyone on the network, can own in minutes. Host-only or internal only.

## A minimal lab, step by step

These are the steps; exact menu labels vary by hypervisor version, so this is a procedure rather
than a click-by-click script.

1. Install the hypervisor.
2. Create a **host-only network** in the hypervisor's network settings. Note its subnet — commonly
   something like `192.168.56.0/24` in VirtualBox's default host-only adapter.
3. Import the **Kali** VM. Give it one adapter on the host-only network. (If you need to update it,
   temporarily add a NAT adapter, update, then remove it.)
4. Import a **target** (start with Metasploitable 2). Give it one adapter on the *same* host-only
   network. Do **not** give it internet access at all.
5. Boot both. From Kali, confirm you can reach the target and that neither can reach the internet:

```bash
# On the attacker VM — find your own address on the host-only net
ip addr show | grep 'inet '

# Confirm the target is reachable (replace with the target's host-only IP)
ping -c 2 192.168.56.101

# Confirm the lab is ISOLATED: this should fail / time out
ping -c 2 8.8.8.8
```

If the last command succeeds, a VM still has a route to the internet — fix the adapter before you
do anything else.

## A note for readers without virtualization

If your machine cannot run VMs comfortably, you can still do most of Level 2 (web testing) with a
single deliberately vulnerable app running locally. For example, DVWA or Juice Shop can be run on
`localhost` directly. The isolation rule is simpler there: you are attacking `127.0.0.1`, which is
your own machine and no one else's. Level 3 (network/infrastructure) genuinely needs target VMs, so
plan to set up virtualization before that level.

## How It Actually Works

A hypervisor presents each VM with a *virtual* network adapter, and that adapter is wired to a
*virtual switch* inside the host. The network "mode" is simply which virtual switch the adapter is
plugged into and whether that switch has an uplink to the real world:

- **Bridged** connects the virtual switch directly to your physical NIC, so the VM gets an address
  on your real LAN and is, for all purposes, another device on your home network.
- **NAT** gives the VM its own private switch whose uplink is a virtual router in the hypervisor
  that masquerades the VM's traffic behind the host's IP — outbound internet works, but inbound is
  blocked, like a home router.
- **Host-only** is a virtual switch with **no uplink at all** — it connects the VMs and the host and
  nothing else. There is physically no path from that switch to the internet, which is what makes
  the isolation reliable rather than a firewall rule you might misconfigure.
- **Internal** is a switch with no uplink *and* no host port — only VMs on it, the most isolated of
  all.

Because host-only has no uplink, the isolation is structural, not a setting that can be bypassed by
a misconfigured guest. That is the whole point: you are not trusting the vulnerable VM to behave;
you have simply not given it a wire to anywhere it shouldn't go. Snapshots work the same way at the
disk layer — the hypervisor records the VM's disk state so that after you compromise and trash a
target, you roll back to a clean snapshot in seconds and attack it again.

## Common mistakes and pitfalls

- **Leaving a vulnerable target on NAT or bridged** "just to download something." Update, then
  remove the internet-facing adapter before you start testing.
- **Running the attacker and target on different, unconnected networks** — then wondering why you
  can't reach the target. They must share the *same* host-only network.
- **Not taking a snapshot before attacking.** You will break targets. Snapshot the clean state
  first so you can reset.
- **Apple-silicon surprises.** Many classic security VMs are x86-64 and run slowly or not at all on
  ARM Macs. Check architecture before committing a lab plan to a specific image.
- **Treating the lab as disposable and sloppy.** Practising good scope discipline (only touch the
  listed targets) even in your own lab builds the habit you need on real engagements.

## Exercise

1. Install a hypervisor and create a host-only network. Write down its subnet.
2. Stand up one attacker VM (Kali or Parrot) and one target (Metasploitable 2 or a Juice Shop VM),
   both on that host-only network.
3. Run the three verification commands from this lesson. Paste the output into your notes and
   confirm: (a) you can reach the target, (b) neither VM can reach `8.8.8.8`.
4. Take a snapshot of the target labelled "clean." In one sentence, explain what you would do to
   reset the target after breaking it, and why snapshots make a lab safe to experiment in.
5. Explain in your own words why host-only isolation is "structural, not a setting" — reference the
   missing uplink.
