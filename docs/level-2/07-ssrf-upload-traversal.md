---
description: "SSRF, insecure file upload and path traversal: tricking the server into fetching, storing or reading resources it should never touch."
---

# SSRF, File Upload & Path Traversal

This lesson groups three flaws that share a theme: the application performs an action on the
*server's* behalf — fetching a URL, saving a file, reading a path — using attacker-controlled input,
and so the attacker makes the *server* do something it shouldn't. Each turns the server into a
confused deputy acting with its own privileges against its own environment. Practise on apps you
run; SSRF especially must never be pointed at systems you don't own.

## Server-Side Request Forgery (SSRF)

SSRF happens when an app fetches a URL that the user controls — "import from URL", a webhook, an
avatar-by-URL, a PDF generator that loads images, a link preview. If the app fetches whatever URL
you give it, you can aim it at targets *it* can reach but *you* cannot.

```
POST /import HTTP/1.1
url=http://169.254.169.254/latest/meta-data/
```

Why that address? In cloud environments, `169.254.169.254` is the **instance metadata service**,
reachable only from the instance itself — and historically a source of temporary credentials. An
SSRF that reaches it can pull cloud credentials. More generally, SSRF lets you:

- scan and reach **internal** services (`http://10.0.0.5:6379/`) behind the firewall;
- hit `http://localhost:…` admin interfaces bound to loopback;
- read cloud metadata;
- use other URL schemes (`file://`, `gopher://`) where the fetcher supports them.

**Testing safely (lab):** stand up a listener you control and give the app a URL pointing at it;
confirm the *server* (not your browser) connects. In the lab you might point it at another lab VM's
internal service to demonstrate reach. Never point an SSRF test at third-party or cloud endpoints
you aren't authorised for.

**Fix:** validate and **allow-list** the destinations the app may fetch (scheme, host, port); resolve
the hostname and re-check the *resolved IP* against the allow-list (to defeat DNS rebinding and
redirect tricks); block link-local/private ranges; disable unneeded URL schemes; don't return the
raw fetched response to the user; and in cloud, require IMDSv2 (session-token metadata) and restrict
the metadata endpoint.

## Insecure File Upload

Upload features are dangerous because a file isn't just data — depending on where it lands and how
it's served, it can become *code*. The worst case: upload a web shell (a script in the app's
language) into a directory the web server executes, then request it — now you run commands on the
server.

What to test on an upload (lab app):

- Does it accept a file whose extension/type shouldn't be allowed (`.php`, `.jsp`, `.aspx`)?
- Can you bypass a weak filter — double extensions (`shell.php.jpg`), case (`.PhP`), null bytes,
  or a spoofed `Content-Type`?
- Is the uploaded file stored *inside the web root* and served with execution? (Upload + reach +
  execute = remote code execution.)
- Can you overwrite existing files (e.g. `.htaccess` to change how a directory is handled)?
- Does image parsing introduce flaws (malicious SVG with script, decompression bombs)?

**Fix:** validate type by content, not just extension/`Content-Type`; use a strict extension
allow-list; store uploads **outside the web root** or on a separate domain served with
`Content-Disposition: attachment` and no execution; randomise stored filenames; set correct
MIME types; scan files; never let uploaded content be served as executable.

## Path Traversal (Directory Traversal)

When an app builds a filesystem path from input — `GET /download?file=report.pdf` → reads
`/var/app/files/report.pdf` — an attacker can inject `../` sequences to climb out of the intended
directory:

```
GET /download?file=../../../../etc/passwd
```

If the server naively joins the input to a base directory, this resolves to `/etc/passwd` and leaks
it. The same flaw lets an attacker read source code, config files with credentials, or SSH keys.

**Testing safely (lab):** request a known-harmless file above the intended directory to prove the
climb works (reading a file you're not meant to reach is the proof; you don't need secrets).

**Fix:** don't build paths from raw input. Resolve the final canonical path and verify it is still
*inside* the permitted base directory before opening it; use an allow-list or an opaque ID that maps
server-side to a filename; strip/reject `..` and encoded variants; run with least privilege so even
a traversal can't read sensitive files.

## A worked example (path traversal, the reasoning)

```python
# UNSAFE
path = os.path.join(BASE_DIR, request.args["file"])
return open(path).read()
```

`os.path.join(BASE_DIR, "../../etc/passwd")` happily produces a path that escapes `BASE_DIR`,
because `join` doesn't constrain the result to the base. The safe version resolves the path and
checks containment:

```python
# SAFE
base = os.path.realpath(BASE_DIR)
target = os.path.realpath(os.path.join(base, request.args["file"]))
if not (target == base or target.startswith(base + os.sep)):
    abort(403)                      # the requested path escaped the base
return open(target).read()
```

`realpath` collapses the `..` sequences to an absolute path; the containment check then rejects
anything that landed outside `base`.

## How It Actually Works

All three flaws are the **confused deputy** problem: a program with more authority than the
requester performs an action *on the requester's behalf*, using the requester's input to decide the
target, without re-checking that the target is one the requester should be allowed to influence. The
server has authority the attacker lacks — it can reach internal hosts, write to the web root, read
the filesystem — and the attacker borrows that authority by controlling the *target* of a privileged
action.

- **SSRF** borrows the server's *network position*: the server can reach `169.254.169.254` and
  internal subnets; the attacker can't, so they supply the URL and let the server make the request
  from inside the trust zone. The response (or its side effects) crosses back out.
- **File upload** borrows the server's *code-execution context*: a file in an executable directory
  runs with the web server's privileges, so controlling the file's content and location converts
  "store my data" into "run my code."
- **Path traversal** borrows the server's *filesystem access*: the process can read many files the
  user can't, so controlling the path reaches them.

The common fix is therefore also one idea: **the privileged action must constrain its target to an
explicit allow-list the attacker cannot expand.** Allow-listed fetch destinations (and re-checking
the resolved IP), allow-listed upload types stored where they can't execute, and paths verified to
stay within a base directory are all the same move — pin down *what the server's authority may act
on* rather than trusting input to name it. Blacklists (block `..`, block `.php`, block
`169.254.*`) fail because the attacker has endless encodings and alternatives; the allow-list
succeeds because it defines the small set of legitimate targets positively.

## Common mistakes and pitfalls

- **Pointing SSRF tests at real cloud metadata or third-party hosts.** That's attacking systems you
  don't own. Use a listener you control or lab-internal targets.
- **Filtering uploads by `Content-Type` or extension alone.** Both are attacker-controlled; bypasses
  are routine. Validate by content and store non-executably.
- **Blacklisting `../` for traversal.** Encoded (`%2e%2e%2f`), doubled, and absolute-path variants
  slip through. Resolve and check containment instead.
- **Returning the SSRF response body to the user** — it turns a blind SSRF into a full internal
  read. (Also a reason attackers love it; a reason defenders should suppress it.)
- **Storing uploads in the web root "for convenience."** This is the single change that turns a
  benign upload bug into RCE. Store outside, serve as attachments.
- **Treating these as low severity.** SSRF-to-metadata and upload-to-RCE are frequently Critical.
  Rate by demonstrated reach/impact.

## Exercise

1. In a lab app with a "fetch URL" feature, start a listener you control (`nc -lvnp 8000` or
   `python3 -m http.server`) and make the app fetch your listener's URL. Confirm the connection
   comes from the *server*, not your browser. Explain why that distinction defines SSRF.
2. On a practice upload form, try to upload a disallowed file type, then a bypass variant (double
   extension or spoofed type). Record what the server accepted and where the file landed.
3. Reproduce path traversal on a lab download endpoint by requesting a harmless file above the
   intended directory. Then apply the `realpath` + containment-check fix and confirm your payload is
   now rejected.
4. For each of the three flaws, write the one-line allow-list fix and explain what target it
   constrains.
5. Explain the "confused deputy" idea in your own words, and why allow-lists beat blacklists for all
   three.
