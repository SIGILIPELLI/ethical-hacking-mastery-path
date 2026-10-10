---
description: "SQL injection explained and demonstrated safely: how string-built queries let input become code, a runnable demo, and the parameterised-query fix."
---

# Injection Flaws & SQL Injection

Injection is the archetypal web vulnerability and a direct consequence of the trust boundary: the
server takes attacker-controlled text and lets it become part of a *command* — an SQL query, a
shell command, an LDAP filter. The most famous form is **SQL injection (SQLi)**, where input
becomes part of a database query. This lesson shows exactly how it happens with a **real,
runnable** demonstration, how to detect it safely in a practice app, and the one fix that actually
works. Practise only against deliberately vulnerable apps you run (DVWA's SQLi labs are purpose-built
for this).

## The root cause: data treated as code

A login handler needs to check a username and password against a database. The *wrong* way builds
the query by gluing strings together:

```python
# UNSAFE — never do this
query = "SELECT id, username, is_admin FROM users " \
        "WHERE username='%s' AND password='%s'" % (username, password)
```

The developer *intended* `username` and `password` to be **data** — values to compare. But by
pasting them into the query string, the input can also contain **SQL syntax**, and the database has
no way to tell the developer's SQL from the attacker's. The data has become code.

## A real demonstration

The following is genuine output from a small SQLite program (`users` table with alice, bob and an
admin). The vulnerable function builds the query by concatenation; the safe one uses parameters.

**Normal login, correct credentials:**

```
query: SELECT id, username, is_admin FROM users WHERE username='alice' AND password='s3cret'
result: [(1, 'alice', 0)]
```

Exactly one row — the intended behaviour.

**Attack — password field set to `' OR '1'='1`:**

```
query: SELECT id, username, is_admin FROM users WHERE username='alice' AND password='' OR '1'='1'
result: [(1, 'alice', 0), (2, 'bob', 0), (3, 'admin', 1)]
```

The injected `' OR '1'='1` closed the password string and added a condition that is *always true*.
The `WHERE` clause now matches **every** row — the login check is bypassed and all users (including
the admin) are returned.

**Attack — username field set to `' OR 1=1 -- `:**

```
query: SELECT id, username, is_admin FROM users WHERE username='' OR 1=1 -- ' AND password='anything'
result: [(1, 'alice', 0), (2, 'bob', 0), (3, 'admin', 1)]
```

Here `' OR 1=1` makes the condition always true, and `-- ` comments out the *rest* of the query
(the entire password check), so the password is never even evaluated. Again every row comes back.

**The same attack against the parameterised (safe) function:**

```
result: []
```

Nothing. No bypass. The fix (below) is why.

## Detecting SQLi safely

You do not need to dump a database to *prove* SQLi exists. Safe indicators, from least to most
intrusive:

1. **Error-based:** submit a single quote `'` in a parameter. A database error in the response
   ("unclosed quotation mark", "syntax error near…") means your quote reached the SQL engine — a
   strong signal.
2. **Boolean-based:** compare `something' AND '1'='1` (should behave normally) with `something' AND
   '1'='2` (should return nothing/different). A behaviour difference confirms your input alters the
   query's logic.
3. **Time-based (blind):** where there's no visible output or error, a payload that makes the
   database *sleep* (e.g. a conditional delay) and a measurable response-time change confirms
   injection without reading any data.

For a professional finding, demonstrating the bypass or a harmless proof (like the database version
via a controlled query) is enough. You rarely need — and the RoE rarely permits — dumping real
customer data. **Prove impact; don't cause it.**

## The fix: parameterised queries (prepared statements)

The input must be sent to the database as **data that can never be parsed as SQL.** That is exactly
what a parameterised query does:

```python
# SAFE
query = "SELECT id, username, is_admin FROM users WHERE username=? AND password=?"
db.execute(query, (username, password))
```

The `?` placeholders are filled by the database driver *after* the SQL has been parsed. The query
structure is fixed; the values are bound separately and treated purely as data. Feed `' OR 1=1 -- `
into the `username` parameter and the database looks for a user literally named `' OR 1=1 -- ` —
finds none — and returns nothing, as the demo showed.

Supporting defences (defence in depth, not substitutes): use an ORM that parameterises by default,
apply least-privilege database accounts, validate input types, and avoid detailed SQL errors in
responses. But **parameterisation is the actual fix** — everything else is a mitigation.

## How It Actually Works

Why does moving the value from the query *string* to a *bound parameter* defeat the attack
completely, rather than just making it harder? It changes *when* the value is interpreted relative
to *when the query is parsed*.

With string concatenation, the database receives one finished string — `SELECT … WHERE username=''
OR 1=1 -- ' AND …` — and its parser reads that string from scratch. The parser has no idea which
characters came from the developer and which from the user; it just sees SQL text and parses it. The
`'` genuinely closes a string literal, `OR 1=1` genuinely becomes a boolean clause, `--` genuinely
starts a comment. The injected characters are *structurally* part of the query because they were
present when parsing happened.

With a parameterised query, the database parses `SELECT … WHERE username=? AND password=?` **first**,
while the placeholders are still empty. Parsing produces a fixed execution plan with two slots for
*values*. Only then does the driver bind the user's input into those slots — and binding does not
re-parse. The bytes `' OR 1=1 -- ` are placed into the value slot as an opaque string; the parser
already finished and will not reconsider them as syntax. There is no path by which a value can become
a clause, because the structural decision (what is syntax) was made before the value existed. That
is why the safe function returned `[]`: it dutifully searched for a user whose name is the literal
string `' OR 1=1 -- `.

This is the general principle behind defeating *all* injection, not just SQL: keep the untrusted
data out of the position where a parser decides structure, and bind it only where values go. Shell
command injection, LDAP injection and NoSQL injection are the same mistake in different
interpreters, and the same separation of code from data is the cure.

## Common mistakes and pitfalls

- **"Escaping" quotes by hand instead of parameterising.** Easy to get wrong (different databases,
  different quoting, Unicode tricks) and it misses non-string contexts. Parameterise.
- **Parameterising values but concatenating table/column names.** Placeholders bind *values*, not
  identifiers. A dynamic column name built from input is still injectable; use an allow-list.
- **Blocking the word `SELECT` or stripping quotes as a "fix".** Blacklist filters are routinely
  bypassed (comments, encoding, case). They are not a fix.
- **Dumping real data to "prove" the bug.** Prove it with an error, a boolean difference, or the DB
  version — not by exfiltrating records you have no need to see.
- **Assuming ORMs are automatically safe.** They parameterise *normal* usage, but raw-SQL escape
  hatches and query-builder misuse reintroduce the flaw.

## Exercise

1. Recreate the demo: build a tiny SQLite `users` table and two functions — one that concatenates
   and one that parameterises. Run the `' OR 1=1 -- ` attack against both and record the results.
   (All local; no network.)
2. In DVWA (or similar), work the SQL Injection lab. Use a single `'` to trigger an error, then a
   boolean pair (`'1'='1` vs `'1'='2`) to confirm the flaw *without* dumping data.
3. Write the vulnerable query your test hit, and rewrite it as a parameterised query. Explain why
   the rewrite defeats your payload.
4. Explain the difference between error-based, boolean-based and time-based detection, and when
   you'd reach for the time-based approach.
5. In your own words, explain *why* binding a parameter after parsing makes injection structurally
   impossible, referencing the order of parse-then-bind.
