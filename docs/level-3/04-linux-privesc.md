---
description: "Linux privilege escalation: turning a limited shell into root via SUID binaries, sudo misconfigurations, cron jobs, capabilities and writable files."
---

# Linux Privilege Escalation

Exploitation usually lands you as a *low-privileged* user — a web-server account, a service user,
`www-data`. Privilege escalation (privesc) is turning that limited foothold into full control
(`root` on Linux). It is almost always a **misconfiguration**, not a memory-corruption exploit:
something runs with more privilege than it should and lets you influence it. This lesson covers the
classic Linux vectors, how to enumerate for them, and — the recurring theme — why each one works.
Practise on lab machines you own.

## The mindset: enumerate, don't guess

Privesc begins with thorough local enumeration — the same "ask politely" philosophy as network
enumeration, now from inside the host. You systematically inspect the system for anything you (the
low-priv user) can influence that runs as root. Automated enumeration scripts (**LinPEAS**,
**linux-smart-enumeration**) collect these signals; you must understand what they're flagging and
why.

Key questions:

- Who am I, and what can I already do? (`id`, `sudo -l`, group memberships)
- What runs as root that I can touch? (SUID binaries, cron jobs, services, writable scripts)
- What's lying around? (readable configs with credentials, SSH keys, history files, backups)
- What's the kernel/OS version? (last resort: a kernel exploit)

## Vector 1 — `sudo` misconfigurations

`sudo -l` lists what your user may run via sudo. Misconfigurations here are the fastest wins:

- A user allowed to run a specific program as root where that program can spawn a shell or read/write
  arbitrary files (many standard binaries have a documented "escape" — see **GTFOBins**).
- `NOPASSWD` entries that let you run powerful commands without even knowing a password.
- Overly broad rules (`ALL`) or editors/interpreters runnable as root (an editor that can shell out,
  a scripting language that can run code).

```bash
sudo -l        # the first thing to run after landing a shell
```

If it says you can run, say, a text editor or an interpreter as root, that program's legitimate
features (shelling out, running code) now run *as root* — instant escalation.

## Vector 2 — SUID / SGID binaries

A **SUID** binary runs with the privileges of its *owner*, not the caller. A SUID-root program runs
as root no matter who launches it. That's intentional for a few programs (`passwd` needs to write a
root-owned file), but a SUID binary that can be made to run arbitrary commands, or read/write
arbitrary files, is an escalation path.

```bash
find / -perm -4000 -type f 2>/dev/null     # list SUID binaries (you saw this in Level 1 lesson 4)
```

Compare the results against **GTFOBins**, which catalogues exactly how common binaries can be abused
when they're SUID or sudo-runnable. A non-standard SUID binary (a custom or misplaced one) is an
immediate red flag.

## Vector 3 — cron jobs and timers

Scheduled tasks often run as root. If a root cron job executes a script that *you* can write to — or
even one in a directory you can write to, or one that calls a program via a relative path you can
hijack via `PATH` — you control code that root will run on the next tick.

```bash
cat /etc/crontab; ls -la /etc/cron.*        # system cron jobs and their scripts
```

Look for a root-run script that is world-writable or in a writable directory. Edit it (in the lab)
to run your command, wait for the schedule, and the command runs as root.

## Vector 4 — Linux capabilities

Capabilities split root's power into pieces that can be granted to individual binaries. A binary with
a dangerous capability (e.g. `cap_setuid`) can be abused to escalate even without the SUID bit:

```bash
getcap -r / 2>/dev/null     # find files with capabilities set
```

An interpreter with `cap_setuid+ep`, for instance, can set its UID to 0 and drop you a root shell.

## Vector 5 — readable secrets & writable sensitive files

- Credentials in config files, scripts, `.bash_history`, environment files, or backups you can read.
- Reused passwords — a database or app password that is also a system user's password.
- World-writable files that matter: a writable `/etc/passwd` (add a root-equivalent user) or a
  writable service unit/script.
- Private SSH keys readable in a home directory — reuse them to access other hosts (lateral
  movement, lesson 7).

## Vector 6 — kernel exploits (last resort)

If the kernel is old and unpatched, a public local-privilege-escalation exploit may work. This is the
*last* resort: kernel exploits can crash the box, they're version-specific, and they're noisy. Prefer
a misconfiguration every time; a crash on a production host is an incident, not a finding.

## A worked example (reasoning)

You land as `www-data` via a web exploit. You run `id` (confirming low privilege), then `sudo -l`
(nothing), then `find / -perm -4000 2>/dev/null`. Among the normal SUID binaries sits an unusual one
— say, a standard utility that GTFOBins lists as exploitable when SUID. You use its documented escape
to spawn a shell; because the binary is SUID-root, the shell runs as root. `id` now shows `uid=0`.
You document: the misconfiguration (that binary should not be SUID-root), the exact steps, the proof
(`id`), the impact (full host compromise), and the fix (remove the SUID bit / the binary).

## How It Actually Works

Why is Linux privesc almost always a misconfiguration rather than a "hack"? Because Unix privilege is
built on a small number of mechanisms that *deliberately* let a process act with more authority than
the user who started it — and each mechanism is safe only if the thing it elevates can't be steered
by an attacker. SUID exists because some legitimate operations (changing your password, binding a
low port) genuinely require root, and the clean way to allow that is a specific program that runs as
root for everyone. The mechanism isn't broken; the bug is granting it to a program that *also* offers
a way to run arbitrary commands or write arbitrary files. The moment a root-privileged context has an
attacker-controllable "do anything" feature, the privilege leaks out through that feature. GTFOBins
is really a catalogue of "standard programs that happen to have a do-anything feature," which is why
making any of them SUID or sudo-runnable is dangerous.

Every vector is the same shape: **a high-privilege action whose *input* or *target* the low-privilege
user can control.** sudo-runnable editor → you control what it opens and it can shell out as root.
World-writable root cron script → you control the code root executes. A capability on an interpreter
→ you control the code that runs with that capability. Writable `/etc/passwd` → you control the user
database root trusts. It is the *confused deputy* again (Level 2 lesson 7), now at the OS level: a
deputy (the SUID binary, cron, the capability) has authority you lack, and the vulnerability is that
you can direct its authority at a target of your choosing. This is why enumeration beats guessing —
you're not looking for a magic exploit, you're looking for the specific place where root's authority
is reachable through something you can write to or influence. And it's why the fixes are mundane and
cheap: remove the SUID bit, tighten the sudo rule, make the cron script non-writable, drop the
capability. You're not patching a bug in code; you're closing a door that was left open.

## Common mistakes and pitfalls

- **Reaching for a kernel exploit first.** It's the loudest, riskiest option. Enumerate for
  misconfigurations first; they're safer and more reliable.
- **Running LinPEAS and not understanding its output.** The script flags candidates; you must know
  *why* each is a path and validate it. Blindly trusting it wastes time on false leads.
- **Ignoring `sudo -l`.** It's the single highest-value first command and people skip it.
- **Not checking GTFOBins** before assuming a SUID/sudo binary is harmless — many innocent-looking
  programs have escapes.
- **Editing a root cron script carelessly** and breaking it (or leaving it broken) — in a real
  engagement that's damage. Note the vector; make minimal, reversible proof changes; clean up.
- **Forgetting lateral reuse.** Credentials and SSH keys found during privesc often unlock *other*
  hosts; capture them for the lateral-movement phase.

## Exercise

1. On a lab Linux target where you have a low-privileged shell, run the enumeration trio: `id`,
   `sudo -l`, and `find / -perm -4000 -type f 2>/dev/null`. Record the output.
2. Run `getcap -r / 2>/dev/null` and inspect `/etc/crontab` and `/etc/cron.*`. Note anything
   root-owned that a low-priv user could influence.
3. Pick one vector present on your target and escalate to root (lab only). Capture the before/after
   `id` output and the exact steps.
4. For your successful vector, write the one-line fix and explain, using the "confused deputy" idea,
   exactly what authority was reachable and how.
5. Explain why enumeration-for-misconfiguration is preferable to a kernel exploit, in terms of
   reliability and risk.
