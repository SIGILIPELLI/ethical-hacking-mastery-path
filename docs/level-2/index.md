---
description: "Ethical Hacking Mastery Path Level 2: web app testing against deliberately vulnerable practice apps — proxies, injection, XSS, access control, APIs."
---

# Level 2 · Web Application Testing

The web is where most real-world findings live, and it is also the easiest place to practise
legally: you can run a deliberately vulnerable app on your own machine and attack it to your
heart's content. This level works entirely against **DVWA** and **OWASP Juice Shop** — apps built
to be broken — so every technique here is something you can reproduce without touching anyone
else's system.

You will learn to think like a web tester: map the app, intercept and modify its traffic, and work
methodically through the OWASP Top 10 classes of flaw — proving each one in a way that shows impact
without destroying data, and noting the fix alongside every finding.

## Modules

1. **[How Web Apps Work & the Testing Mindset](01-how-web-apps-work.md)** — requests, responses,
   state, and where trust boundaries sit.
2. **[Using an Intercepting Proxy](02-intercepting-proxy.md)** — Burp Suite and OWASP ZAP: capture,
   modify and replay requests.
3. **[Injection Flaws & SQL Injection](03-sql-injection.md)** — how untrusted input becomes code,
   demonstrated safely in DVWA, and the parameterised-query fix.
4. **[Cross-Site Scripting (XSS)](04-xss.md)** — reflected, stored and DOM XSS, impact, and output
   encoding + CSP as the defence.
5. **[Broken Authentication & Sessions](05-broken-authentication.md)** — credential handling,
   session fixation, and rate limiting.
6. **[Broken Access Control & IDOR](06-access-control-idor.md)** — the most common serious flaw:
   acting outside your permissions.
7. **[SSRF, File Upload & Path Traversal](07-ssrf-upload-traversal.md)** — tricking the server into
   fetching, storing or reading what it should not.
8. **[Misconfiguration & Vulnerable Components](08-misconfiguration-components.md)** — defaults,
   headers, and outdated dependencies.
9. **[API Security Testing](09-api-security-testing.md)** — REST and GraphQL endpoints, auth,
   object-level authorization and rate limits.
10. **[Project — Assessing OWASP Juice Shop](10-project-juice-shop.md)** — a structured assessment
    with a written findings report.
