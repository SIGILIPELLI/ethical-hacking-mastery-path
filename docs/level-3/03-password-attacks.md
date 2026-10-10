---
description: "Password attacks and cracking: hashing vs encryption, dictionary and brute-force attacks, and why salting and slow hashes defeat them."
---

# Password Attacks & Cracking

Passwords are everywhere and routinely the weakest link. This lesson covers how passwords are stored
(hashing, not encryption), how attackers recover them (online guessing and offline cracking), and —
most importantly — *why* modern storage (salting plus a deliberately slow hash) makes cracking
impractical. Every demonstration here cracks **hashes you generate yourself**; never attack hashes
or accounts you don't own.

## Hashing vs encryption — a critical distinction

Passwords should be **hashed**, not encrypted.

- **Encryption** is reversible: with the key, ciphertext → plaintext. If an app can decrypt its
  stored passwords, so can an attacker who steals the key.
- **Hashing** is one-way: `hash(password)` → a fixed-length digest that cannot be reversed to the
  password. To verify a login, the app hashes the submitted password and compares digests; it never
  needs the original.

So a breached password database should contain *hashes*. An attacker can't reverse a hash — but they
can **guess**: hash a candidate and see if it matches a stolen digest. That is what "cracking" is.

## Online vs offline attacks

- **Online** — guessing against a live login (Level 2 lesson 5). Slow and noisy; defeated by rate
  limiting and lockout. Techniques: credential stuffing, password spraying.
- **Offline** — the attacker has the *hashes* (from a breach, a dumped database, a captured
  Windows SAM/NTDS) and cracks them on their own hardware at full speed, with no rate limit to stop
  them. This is where `hashcat` and `john` (John the Ripper) live, and it's why *how* a password was
  hashed matters enormously.

## A real cracking demonstration

Below is genuine output. A "target" chose the password `Summer2024`. The attacker has only the
stored hashes and runs a small dictionary (common words × capitalisation × year/suffix "mangling
rules").

**Unsalted MD5 and salted SHA-256:**

```
Captured MD5 hash:        e90664c0af74160644d29e4d6147969b
Captured salted SHA-256:  a094458cbf02745eec99901b1a58a653ecf1acb9f50ae7f1f2ff93a46cfe01bb  (salt: a1b2c3d4)

Dictionary attack vs MD5:
  tried 27 candidates in 0.0000s -> cracked: 'Summer2024'
Dictionary attack vs salted SHA-256 (salt known):
  tried 27 candidates in 0.0000s -> cracked: 'Summer2024'
```

Twenty-seven guesses, instant. `Summer2024` looks "complex" (capital, digits) but it is a dictionary
word plus a year — exactly what mangling rules generate first. And note the salted SHA-256 fell just
as fast: **salting does not slow down cracking a single known hash** (more on what salting *does* do
below). The real problem here is that MD5 and SHA-256 are *fast* — billions of guesses per second on
a GPU.

**Now the same attack against a slow hash (PBKDF2, 600,000 iterations):**

```
One PBKDF2 guess (600,000 iters): 72.3 ms
Fast MD5 guess: ~0.00390 ms
Slowdown factor: ~18,537x per guess
=> A 27-word list: 1.95s (PBKDF2) vs <0.001s (MD5)
=> A 14-million-word list would take ~281 hours on this one core.
```

The password didn't change; the *storage* did. Each guess now costs ~72 ms instead of microseconds
— an ~18,500× slowdown measured on this machine. A wordlist that cracks the MD5 instantly would take
hundreds of hours against PBKDF2 on one core. That is the entire defence: make each guess expensive.

## What salting actually does

Salting adds a unique random value to each password before hashing. From the demo, salting didn't
slow cracking *one* known hash — so why do it? Salting defeats **precomputation and bulk attacks**:

- **Rainbow tables** (giant precomputed hash→password lookups) become useless: the table would have
  to be rebuilt for every possible salt.
- **Identical passwords get different hashes**, so an attacker who cracks one doesn't instantly crack
  every other user with the same password, and can't see which users share a password.
- The attacker must attack **each hash separately** rather than testing one guess against the whole
  database at once.

So the modern recipe is **unique salt per password** (defeats precomputation and bulk) **plus a slow
hash** (defeats fast guessing). You need both.

## The tools (lab use)

- **hashcat** — GPU-accelerated; extremely fast across many hash types. Modes: dictionary (`-a 0`
  with a wordlist and rules), brute-force/mask (`-a 3`), combinator.
- **John the Ripper** — CPU-focused, great at autodetecting formats and at Windows hashes.
- **Wordlists** — `rockyou.txt` (the classic ~14M leaked-password list) and others; **rules** mangle
  base words (capitalise, append years, l33t-speak) to multiply coverage.

```bash
# Crack YOUR OWN generated hashes in the lab (example invocations):
hashcat -m 0 -a 0 my_md5_hashes.txt rockyou.txt            # -m 0 = MD5
john --format=raw-sha256 --wordlist=rockyou.txt my_hashes  # John, SHA-256
```

(hashcat/john ship with Kali. The exact crack speed depends entirely on your hardware; this course
does not quote GPU benchmarks it didn't run — generate hashes and measure your own.)

## The defences

- **Store passwords with a slow, salted, adaptive hash:** **argon2id** (preferred), **bcrypt**, or
  **scrypt**; PBKDF2 with a high iteration count where those aren't available. Never MD5, SHA-1, or
  plain SHA-256 for passwords.
- **Unique salt per password** (the hash functions above handle this for you).
- **Tune the cost** so one hash takes a meaningful fraction of a second, and raise it as hardware
  improves (that "adaptive" property is why bcrypt/argon2 have a cost parameter).
- **Encourage length over complexity** — a long passphrase beats `P@ss1!`; block known-breached
  passwords; offer MFA so a cracked password alone isn't enough.
- **Protect the hashes** in the first place — a breach that never happens needs no cracking.

## How It Actually Works

Why does making a hash *slow* defeat cracking, when the attacker has the same hash function you do?
Because cracking is a numbers game measured in *guesses per second*, and the defender gets to set the
price of a guess. A fast hash like MD5 was designed to be cheap — it's meant for checksums over large
files, so hardware can compute billions per second. When that same speed is applied to password
guessing, the attacker's throughput is astronomical and any human-memorable password falls. A slow
hash deliberately inverts this: argon2 and bcrypt perform many internal rounds (and argon2/scrypt
also consume lots of *memory*, which GPUs can't parallelise cheaply), so a single hash costs
milliseconds by design. The measured ~18,500× slowdown isn't an accident — it's the cost parameter
doing exactly its job. Multiply a per-guess cost of 72 ms across the billions of guesses needed to
exhaust a real keyspace, and the attack moves from "minutes" to "centuries." The password's strength
didn't change between the two demos; the *economics of guessing* did.

Salting works at a different point in the economics. Hashing is deterministic — the same input always
gives the same digest — which is what lets an attacker *precompute*: hash every likely password once,
store the results, and then "crack" any fast-hashed database by lookup, paying the hashing cost only
once for all victims ever. A unique salt per password breaks determinism *across users*: `hash(salt_A
+ pw)` and `hash(salt_B + pw)` differ even for the same `pw`, so a precomputed table built for one
salt is worthless for another, and the attacker must redo the work for every single hash. Salting
doesn't make one guess slower (the demo showed that); it destroys the *amortisation* that made bulk
and precomputed attacks cheap. Slow hashing raises the per-guess price; salting forces the attacker
to pay it separately for every account. Together they turn a stolen database from "cracked over
lunch" into "economically infeasible," which is the realistic goal — you can't stop guessing, you can
only make it cost more than the result is worth.

## Common mistakes and pitfalls

- **Hashing passwords with MD5/SHA-1/SHA-256.** Fast hashes; wrong tool. Use argon2id/bcrypt/scrypt.
- **"Encrypting" passwords.** Reversible storage means a stolen key reveals everything. Hash, don't
  encrypt.
- **Omitting salt, or using one global salt.** Re-enables rainbow tables and bulk cracking. Unique
  salt per password.
- **Equating "complex" with "strong".** `Summer2024!` is cracked early by mangling rules; length and
  unpredictability matter more.
- **Cracking hashes you don't own.** Only ever crack hashes you generated or are explicitly
  authorised to, inside scope.
- **Relying on password rules instead of MFA + breach-list checks.** Modern guidance favours length,
  blocking breached passwords, and a second factor over arbitrary complexity rules.

## Exercise

1. Reproduce the cracking demo: generate an MD5 and a salted SHA-256 of a password *you* choose, then
   write a small dictionary + mangling attack and crack them. Record how many candidates it took.
2. Reproduce the slow-hash demo: hash the same password with PBKDF2 (e.g. 600k iterations via your
   language's stdlib) and *measure* one guess's time on your machine. Compute your own slowdown
   factor vs. MD5.
3. Explain, from your measurements, why the slow hash defeats the wordlist that instantly cracks the
   MD5 — even though the password is identical.
4. In your own words, state precisely what salting does and does *not* protect against, using the
   "precomputation/amortisation" idea.
5. Pick the correct storage scheme for a new app and justify it: which algorithm, salted how, and how
   you'd choose the cost parameter.
