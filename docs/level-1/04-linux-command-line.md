---
description: "The Linux command line a penetration tester relies on: navigation, file and text tools, pipes, grep, permissions, processes and networking."
---

# The Linux Command Line You'll Actually Use

Almost every offensive-security tool runs on Linux, most of them on the command line, and most of
them emit text you then have to filter, sort and search. You do not need to be a kernel hacker, but
you do need to be fluent enough that the shell gets out of your way. This lesson is the practical
subset a tester leans on daily. If you already know Linux well, skim it and move on; if you don't,
type every command.

Everything here runs on your attacker VM (or any Linux machine, including WSL). The commands below
are real and reproducible on a standard Linux shell.

## Moving around and looking at files

```bash
pwd                 # where am I
ls -la              # list everything, long form, including hidden files
cd /etc             # change directory
cat /etc/hostname   # print a whole file
less /var/log/syslog  # page through a large file (q to quit)
head -n 20 file     # first 20 lines
tail -n 20 file     # last 20 lines
tail -f file.log    # follow a log as it grows
```

## Finding things

```bash
find / -name "*.conf" 2>/dev/null     # find files by name, hide permission-denied noise
find / -perm -4000 2>/dev/null        # find SUID binaries (you'll use this in privesc)
which nmap                            # where is a command
locate wordlist                       # fast filename search (needs updatedb)
grep -r "password" /etc 2>/dev/null   # recursively search file contents
```

That `2>/dev/null` sends error output (file descriptor 2, stderr) to nowhere, so a `find /` doesn't
bury its real results under thousands of "Permission denied" lines. Understanding that `>` redirects
output and `2>` redirects errors is worth more than it looks.

## Pipes and text processing — the real skill

The power of the shell is composing small tools with the pipe `|`, which feeds one command's output
into the next. Tool output is almost always text, and shaping text is most of the work:

```bash
# How many lines did nmap find mentioning 'open'?
cat scan.txt | grep open | wc -l

# Pull just the IP column from a file and sort unique
cat hosts.txt | cut -d' ' -f1 | sort -u

# Extract the second field of a colon-separated file (e.g. a hash file)
cut -d: -f2 hashes.txt

# Replace text on the fly
echo "admin:changeme" | sed 's/changeme/REDACTED/'

# Field-aware processing with awk: print user and shell from /etc/passwd
awk -F: '{print $1, $7}' /etc/passwd | head
```

A concrete, runnable example — list the login shells in use on the system, most common first:

```bash
awk -F: '{print $7}' /etc/passwd | sort | uniq -c | sort -rn
```

This reads `/etc/passwd`, pulls the 7th colon-separated field (the shell), sorts them, counts each
unique value with `uniq -c`, and sorts the counts descending. That pattern — `sort | uniq -c | sort
-rn` — is how you find "the most common thing" in any list, and you will use it constantly on scan
output.

## Permissions, users and processes

```bash
id                  # who am I, what groups
whoami
sudo -l             # what can I run as root (key privesc check)
chmod +x script.sh  # make a file executable
chown user:group f  # change ownership
ps aux              # all running processes
ps aux | grep ssh   # is sshd running
kill 1234           # terminate process 1234
```

Linux permissions are read/write/execute for owner, group and others — the `rwxr-xr--` string in
`ls -l`. A tester reads these constantly: a world-writable config, a script owned by root but
editable by everyone, a SUID binary (runs as its owner regardless of who calls it). These are the
raw material of privilege escalation (Level 3).

## Networking from the shell

```bash
ip addr             # my interfaces and addresses (modern replacement for ifconfig)
ip route            # my routing table
ss -tlnp            # listening TCP ports on THIS machine (replaces netstat)
curl -I http://target/            # fetch just HTTP headers
curl http://target/robots.txt     # fetch a file
wget http://target/file           # download a file
dig example.com                   # DNS lookup
nc -nv 127.0.0.1 80               # connect to a port by hand (netcat)
```

## Making life bearable

```bash
history             # commands you've run (your audit trail — keep it)
!!                  # repeat the last command
command | tee out.txt   # see output AND save it to a file
Ctrl-R              # reverse-search your history as you type
```

`tee` matters for testers: it lets you watch a tool's output live *and* keep a copy for the report.
Your shell history and saved output are part of your evidence trail.

## How It Actually Works

Why is "a pile of small text tools joined by pipes" so much more useful than one big program? Two
design decisions in Unix:

1. **Everything is a stream of text.** A command's standard output is just bytes going to a file
   descriptor (fd 1). A pipe `a | b` connects `a`'s fd 1 to `b`'s standard input (fd 0). Neither
   command knows or cares that the other exists — the shell wires their descriptors together and the
   kernel buffers the bytes between them. Because the interface between tools is *just text*, any
   tool can feed any other. That is why `nmap`'s output can flow into `grep` into `cut` into `sort`
   even though their authors never coordinated.

2. **Each tool does one thing.** `grep` only filters lines, `sort` only sorts, `wc` only counts.
   Because none of them tries to do everything, you compose exactly the pipeline your problem needs
   rather than hoping one monolith has the feature. A tester's filtering needs are endless and
   unpredictable, so composability beats any fixed feature set.

The three file descriptors (0 stdin, 1 stdout, 2 stderr) being separate is why `2>/dev/null` can
throw away errors while keeping results: the tool writes results to fd 1 and complaints to fd 2, and
you route them independently. Once you internalise "stdout is results, stderr is diagnostics, pipes
join stdout to the next stdin," the whole shell stops being magic and becomes plumbing you control.

## Common mistakes and pitfalls

- **Forgetting `2>/dev/null` on `find /`** and drowning real hits in permission errors.
- **Parsing columns with `cut -d' '` when fields are separated by *multiple* spaces.** Use `awk`,
  which treats runs of whitespace as one separator by default.
- **Running everything as root in the lab "to avoid permission issues."** You then learn nothing
  about the permission model that privilege escalation exploits. Work as a normal user; use `sudo`
  deliberately.
- **Not saving output.** You run a great scan, close the terminal, and it's gone. Pipe through
  `tee` or redirect to a file — the report needs it.
- **Confusing `>` (overwrite) with `>>` (append).** `>` silently clobbers an existing file.

## Exercise

1. Using only `awk`, `sort`, `uniq` and `grep`, write a one-line pipeline that prints the three most
   common login shells on your system with their counts. (Hint: it is in this lesson.)
2. Run `find / -perm -4000 2>/dev/null` on your attacker VM and save the output with `tee
   suid.txt`. You will revisit this list in Level 3.
3. Use `ss -tlnp` to list the listening ports on your own machine. Pick one and explain what service
   you think is behind it.
4. Explain, in your own words, what the `|` in `a | b` actually connects, referencing file
   descriptors 0 and 1.
5. Create a file with `echo "line1" > f.txt`, then append with `echo "line2" >> f.txt`, then try
   `echo "oops" > f.txt` and `cat f.txt`. Explain what happened to line1 and line2.
