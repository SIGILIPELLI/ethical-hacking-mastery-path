---
description: "Security misconfiguration and vulnerable components: default settings, missing security headers, verbose errors, and outdated dependencies with known CVEs."
---

# Misconfiguration & Vulnerable Components

Not every web finding is a clever injection. Two of the most common OWASP categories are, frankly,
"things left wrong": **Security Misconfiguration** (defaults, unnecessary features, missing
hardening) and **Vulnerable and Outdated Components** (using a library or server version with known
CVEs). They are unglamorous, extremely widespread, and often high-impact — a default admin password
or an outdated component with a public exploit can be the whole breach. This lesson is how to find
and report them. Test on apps you run.

## Security misconfiguration — what to look for

### Default and weak configuration

- **Default credentials** — admin consoles, databases, devices shipped with `admin/admin`,
  documented defaults, or setup accounts never removed. Check the vendor's docs for defaults and try
  them (on your own lab systems).
- **Default/sample content** — example apps, test pages, setup scripts, or admin panels left
  enabled (`/phpmyadmin`, `/manager/html`, framework debug consoles).
- **Unnecessary features** — services, ports, HTTP methods (`PUT`, `TRACE`), or modules enabled but
  unused, each adding attack surface.

### Information disclosure

- **Verbose errors** — stack traces, SQL errors, framework debug pages that reveal file paths,
  versions, and internal structure. The debug toolbar enabled in production is a classic.
- **Directory listing** — the web server showing a directory's files when there's no index page.
- **Exposed files** — `.git/` directories, backup files (`config.php.bak`), `.env` files,
  `phpinfo()` pages, Swagger/OpenAPI docs that weren't meant to be public.

### Missing security headers

Response headers that harden the browser's behaviour; their absence is a reportable weakness:

| Header | Protects against |
|--------|-------------------|
| `Content-Security-Policy` | XSS, data injection (restricts script sources) |
| `Strict-Transport-Security` | Protocol downgrade / SSL stripping |
| `X-Content-Type-Options: nosniff` | MIME-sniffing attacks |
| `X-Frame-Options` / CSP `frame-ancestors` | Clickjacking |
| `Set-Cookie` flags (`HttpOnly`, `Secure`, `SameSite`) | Cookie theft / CSRF |

Check them directly:

```bash
curl -sI https://app.lab/ | grep -iE 'content-security|strict-transport|x-content-type|x-frame|set-cookie'
```

Missing lines are candidate findings (weighted by the app's exposure and whether other defences
compensate).

### TLS and transport

- Is HTTPS enforced (HTTP redirects to HTTPS; HSTS set)?
- Weak/old TLS versions or ciphers enabled? (Tools like `testssl.sh` check this against your own
  hosts.)
- Mixed content (HTTPS page loading HTTP resources)?

## Vulnerable and outdated components

Modern apps are mostly other people's code — frameworks, libraries, the web server, the runtime.
Any of them can have a known CVE. This is the category behind some of the largest breaches (an
unpatched framework, a logging library with a remote-code-execution bug).

How to assess it:

1. **Inventory the components and versions.** From banners (`Server`, `X-Powered-By`), from
   JavaScript bundle contents, from fingerprinting (`whatweb`), and — if you have the source in the
   lab — from the dependency manifests (`package.json`/`package-lock.json`, `requirements.txt`,
   `pom.xml`).
2. **Check each against known vulnerabilities.** The NVD, vendor advisories, GitHub Advisory
   Database, and **automated scanners**:
   - `npm audit` (Node), `pip-audit` (Python), `mvn dependency-check` / OWASP Dependency-Check
     (Java) — these read your manifests and report known-vulnerable versions.
3. **Validate reachability.** A vulnerable dependency that is present but never exercises the
   vulnerable code path is lower risk than one directly reachable from input. Note the distinction in
   the report (ties back to Level 1's CVE/CVSS lesson).

```bash
# In a lab project you have the source for:
npm audit                      # Node dependencies
pip-audit                      # Python dependencies (installs via pip)
```

These read the lockfile/environment and list advisories with their severities. Treat the output as
*candidates* and confirm the vulnerable version is actually in use and reachable.

## A worked example

Fingerprinting a lab app shows an old, EOL version of a web framework in the `X-Powered-By` header,
and directory listing is enabled on `/uploads/`. Your findings:

1. **Outdated framework (version X).** Cross-reference the NVD: the version is affected by a
   High-severity CVE with a public exploit. Validate that the vulnerable feature is reachable.
   Finding: outdated component with known RCE; remediation: upgrade to the fixed release.
2. **Directory listing on `/uploads/`.** Browsing it reveals every uploaded file, including other
   users' documents. Finding: information disclosure via misconfiguration; remediation: disable
   auto-indexing, add access control.

Each is written with evidence (the header, the directory contents), severity, and a concrete fix.

## The defences

- **Harden by default.** A repeatable, hardened build/deploy process; no manual snowflake configs.
  Remove sample apps, disable unused features and methods, change every default credential.
- **Suppress verbose errors in production**; log details server-side, show users a generic message.
- **Set security headers** site-wide; enforce HTTPS + HSTS.
- **Disable directory listing**; keep `.git`, backups and `.env` out of the web root.
- **Maintain a dependency inventory (SBOM)** and patch on a schedule; wire dependency scanning into
  CI so new CVEs surface automatically; remove unused dependencies.

## How It Actually Works

Why are misconfiguration and outdated components so stubbornly common when the fixes are usually
trivial? Because both are failures of *process*, not of code, and processes decay silently. A
default credential isn't a bug a developer wrote; it's a step someone didn't do. Directory listing
isn't code; it's a web-server default nobody turned off. An outdated library isn't a mistake in the
app; it's the passage of time — the version was fine when shipped and became vulnerable when a CVE
was published later. None of these produce a failing test or a visible error during development, so
nothing surfaces them. The app works perfectly; it is merely *exposed*. That invisibility is exactly
why they survive to production and why scanning and inventory matter so much: you cannot
notice-by-using what only a deliberate check reveals.

The dependency problem has a structural edge worth understanding: your security is now a function of
the security of code you didn't write and may not even know you depend on (transitive
dependencies — your dependency's dependencies). A single widely-used library with a critical bug
instantly makes millions of apps vulnerable the day the CVE drops, with no action by any attacker
yet. This is why the industry moved toward SBOMs (software bills of materials) and automated
advisory feeds: the only way to manage risk you inherited from an ecosystem is to *inventory* what
you inherited and *subscribe* to news about it, so that when a component's status flips from "fine"
to "vulnerable," you find out from your pipeline rather than from your incident responders. The fix
being easy (upgrade, flip a setting, delete a default) is precisely why these are satisfying
findings to report: high impact, low remediation cost.

## Common mistakes and pitfalls

- **Reporting every missing header as High.** Context matters — a missing CSP on a static brochure
  site is minor; on an app with user content it's serious. Rate by exposure and compensating
  controls.
- **Trusting `npm audit`/scanner output as confirmed and critical.** It lists advisories by
  theoretical severity; validate that the vulnerable version and code path are actually in use.
- **Trying default creds against systems you don't own.** Lab/authorised only.
- **Missing exposed `.git`/backup/`.env` files** because you only tested the linked pages. Probe for
  them explicitly.
- **Only inventorying direct dependencies.** Transitive dependencies carry CVEs too; use tools that
  walk the full tree.
- **Confusing "present" with "exploitable" for a component CVE.** Note reachability; it changes the
  severity and the client's prioritisation.

## Exercise

1. Run `curl -sI` against a practice app and list which security headers are present and which are
   missing. For each missing one, name the attack it would have hindered.
2. Probe a lab app for exposed artefacts: try `/.git/`, a backup filename, `/.env`, and a known
   admin/sample path. Record anything that responds.
3. In a lab project with source, run `npm audit` or `pip-audit`. Pick one reported advisory and
   determine whether the vulnerable version is actually in use and reachable.
4. Fingerprint a lab app's framework/server version and look up one CVE affecting it. Write it as a
   finding with evidence, severity and remediation.
5. Explain, in your own words, why misconfiguration and outdated-component flaws are "process
   failures that decay silently," and why they don't show up by simply using the app.
