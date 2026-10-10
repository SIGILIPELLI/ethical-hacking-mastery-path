---
description: "Windows privilege escalation: service misconfigurations, unquoted service paths, token privileges, registry autoruns and stored credentials."
---

# Windows Privilege Escalation

The Linux privesc lesson's lesson transfers directly: escalation on Windows is also almost always a
**misconfiguration** — a high-privilege context you can influence — and the goal is the equivalent of
root: **NT AUTHORITY\SYSTEM**. The mechanisms differ (services, tokens, the registry, the Windows
permission model), so this lesson maps the Windows-specific vectors and how to enumerate them.
Practise on lab Windows VMs you own.

## The privilege landscape

- **Standard user** → **Administrator** → **SYSTEM**. Administrators can do almost anything but run
  at Medium/High integrity; SYSTEM is the OS's own account and the usual end goal.
- **UAC (User Account Control)** means even an admin's processes run unprivileged by default until
  elevated — so "I'm an admin" and "I have an elevated shell" are different, and UAC bypass is its
  own topic.
- **Integrity levels** (Low/Medium/High/System) and **privileges** (named rights like
  `SeImpersonatePrivilege`) are the fine-grained controls that the vectors below abuse.

## Enumeration first

As on Linux, enumerate thoroughly before acting. **WinPEAS**, **Seatbelt** and PowerShell scripts
(**PowerUp**) automate the signal collection. Core manual checks:

```text
whoami /priv              # YOUR privileges — SeImpersonate, SeBackup, etc.
whoami /groups            # group memberships
systeminfo                # OS version and hotfixes (for missing-patch analysis)
net user / net localgroup administrators
```

## Vector 1 — service misconfigurations

Windows services often run as SYSTEM. If a standard user can modify such a service, they can make
SYSTEM run their code:

- **Weak service permissions** — you can change a SYSTEM service's binary path (`binPath`) to your
  own program, then restart it.
- **Writable service executable** — the service's `.exe` is in a directory you can write to; replace
  or hijack it.
- **Unquoted service path** (below) — a classic.

Tools like `sc qc <service>`, `accesschk`, and PowerUp's checks surface these.

## Vector 2 — unquoted service paths

If a service's executable path contains spaces and is **not quoted**, Windows' path-resolution tries
each space-delimited prefix with `.exe` appended. For a path like
`C:\Program Files\My App\service.exe` (unquoted), Windows tries, in order:

```
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My App\service.exe
```

If you can write `C:\Program.exe` (or `C:\Program Files\My.exe`), Windows runs *your* binary as the
service account (often SYSTEM) before it ever reaches the real one. The fix is simply quoting the
path — which is why this is a configuration bug, not a code bug.

## Vector 3 — token privileges (the "Potato" family)

Certain privileges held by service accounts are dangerous:

- **`SeImpersonatePrivilege`** — held by many service accounts (IIS, SQL Server). It allows
  impersonating another token, and a family of public techniques ("Potato" exploits) abuse it to
  obtain a SYSTEM token. If `whoami /priv` shows `SeImpersonatePrivilege` **Enabled**, that is often
  the path.
- **`SeBackupPrivilege` / `SeRestorePrivilege`** — read/write any file, e.g. to grab the SAM and
  SYSTEM registry hives (which contain local password hashes) or overwrite protected files.
- **`SeDebugPrivilege`** — debug (and inject into) other processes, including SYSTEM ones.

The presence of these in `whoami /priv` is the signal; the technique depends on which one.

## Vector 4 — registry & autoruns

- **AutoRun / startup programs** stored in registry keys or startup folders you can write to — your
  payload runs when a higher-privileged user (or the system) next starts.
- **`AlwaysInstallElevated`** — if both the HKLM and HKCU policy keys are set, any user can install
  an MSI package as SYSTEM. A straightforward escalation when present.
- **Weak registry key permissions** on service configuration keys — another route to altering how a
  SYSTEM service launches.

## Vector 5 — stored credentials

Windows is full of places credentials hide:

- **Credential Manager**, saved RDP credentials, the SAM/NTDS hashes (with the right privilege).
- **Unattended install files** (`Unattend.xml`, `sysprep.xml`) sometimes contain plaintext admin
  passwords.
- **Scripts, config files and the registry** with embedded passwords.
- **Reused passwords** across local admin accounts (a single cracked local-admin password that is
  identical everywhere is the mechanism behind sweeping lateral movement — mitigated in managed
  environments by per-machine random local-admin passwords).

## A worked example (reasoning)

You have a standard-user shell on a lab Windows box. `whoami /priv` shows `SeImpersonatePrivilege:
Enabled` (you landed via a web/service account). That single line tells you the path: a
`SeImpersonate`-based technique to obtain a SYSTEM token. You run the appropriate tool in the lab,
confirm `whoami` now returns `nt authority\system`, and document: the privilege that enabled it, the
steps, the proof, the impact (full host control), and the fix (the service shouldn't hold that
privilege, or the service account should be constrained). Had `whoami /priv` instead been
unremarkable, you'd pivot to service-permission and unquoted-path checks via PowerUp/WinPEAS.

## How It Actually Works

Why does Windows privesc, despite a completely different OS design, reduce to the *same* principle as
Linux — "a high-privilege action whose input or target you control"? Because privilege escalation is
not really an OS-specific trick; it's a property of any system that lets some processes act with
more authority than the user who launched them. Windows has *more* such mechanisms than Linux
(services as SYSTEM, a rich privilege model, the registry as a system-wide configuration store,
impersonation tokens), which means *more surfaces* where authority can be reached — but each is the
confused-deputy pattern again. An unquoted service path lets you control *which file* SYSTEM
executes. Weak service permissions let you control *what the service runs*. `SeImpersonatePrivilege`
lets a service account control *whose token it adopts* — and token impersonation exists for a
legitimate reason (a service handling a client request needs to act as that client), so the
"vulnerability" is a legitimate feature reachable from a context an attacker occupies.
`AlwaysInstallElevated` literally delegates SYSTEM-level installation to any user by policy.

The token model is the most Windows-specific idea and worth internalising: in Windows, a process's
privileges are carried by an *access token* attached to it, and impersonation is the designed ability
for a process to temporarily wear another token. SYSTEM processes have the most powerful tokens, so
the whole game of the "Potato" techniques is to coerce a SYSTEM-level service into authenticating to
something you control, capture or impersonate the resulting token, and thereby wear SYSTEM's
authority. Nothing is "broken" in the cryptographic sense — you've arranged for a privileged deputy
to hand you, or act under, its credentials. That's why the defences are, once again, configuration
and least privilege: quote service paths, lock down service and registry permissions, strip
unnecessary privileges from service accounts, don't set `AlwaysInstallElevated`, and never leave
credentials in files. You close the specific door; you don't patch a flaw in the kernel.

## Common mistakes and pitfalls

- **Confusing "admin" with "elevated".** Thanks to UAC, an admin's default shell isn't elevated.
  Know which you have before concluding you've won.
- **Skipping `whoami /priv`.** It frequently names the escalation path outright
  (`SeImpersonatePrivilege`, `SeBackupPrivilege`). It's the Windows equivalent of `sudo -l`.
- **Running WinPEAS without understanding it.** It surfaces candidates; you validate and understand
  each, as on Linux.
- **Replacing a service binary and forgetting cleanup/consequences.** You may break the service; in a
  real engagement that's damage. Make minimal, reversible proofs and restore state.
- **Missing unquoted paths because the real exe exists.** The real binary at the end of the path
  still runs *normally* — the bug is that a writable earlier prefix runs *first*. Check writability of
  the prefixes, not just whether the service works.
- **Treating hashes as needing cracking.** A captured NTLM hash can often be *used directly*
  (pass-the-hash, next lessons) without ever cracking it.

## Exercise

1. On a lab Windows VM with a standard-user shell, run `whoami /priv`, `whoami /groups` and
   `systeminfo`. Record the privileges you hold.
2. Enumerate services for weak permissions and unquoted paths (via `sc`, `accesschk`, or PowerUp).
   List any service a standard user could influence.
3. If your lab box exposes one, escalate to SYSTEM via a single vector (lab only). Capture `whoami`
   before and after and the steps.
4. Explain the unquoted-service-path mechanism in your own words, including why the real executable
   still existing doesn't prevent the attack.
5. Explain why a captured NTLM hash may not need cracking, and connect that to why least-privilege on
   service accounts is a key Windows defence.
