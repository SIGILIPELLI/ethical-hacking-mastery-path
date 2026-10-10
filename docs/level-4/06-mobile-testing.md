---
description: "Mobile application testing basics: the Android and iOS attack surface, insecure storage, traffic interception and certificate pinning, and the OWASP MASVS."
---

# Mobile Application Testing Basics

Mobile apps have a different attack surface from web apps: the client runs on a device the user (and
attacker) fully controls, stores data locally, talks to backends over the network, and uses
platform APIs. A mobile pentest examines the app on the device, its local storage, its network
traffic, and its backend. This lesson maps that surface for Android and iOS, the characteristic
weaknesses, and the tooling — grounded in the OWASP **Mobile Application Security Verification
Standard (MASVS)** and **MASTG** testing guide. Test only apps you own or are authorized to assess,
on devices/emulators you control.

## Why mobile is different

- **The client is on hostile hardware.** Like web (Level 2 lesson 1), the client is attacker-
  controlled — but *more so*: the attacker can root/jailbreak the device, decompile the app, hook its
  functions at runtime, and read its storage. Any secret or logic shipped in the app is recoverable.
- **Local data persistence.** Apps store data on the device (databases, preferences, files, caches).
  How securely they do so is a major test area with no web equivalent.
- **Platform APIs & IPC.** Inter-process communication (Android intents, content providers; iOS URL
  schemes, extensions) is an attack surface.
- **The backend is still there.** A mobile app is a client to APIs — so Level 2's web/API testing
  (injection, BOLA, auth) applies to the backend in full.

## The two platforms

| Aspect | Android | iOS |
|--------|---------|-----|
| Package | APK/AAB | IPA |
| Language | Kotlin/Java (+ native) | Swift/Objective-C |
| Local storage | SharedPreferences, SQLite, files | UserDefaults, Keychain, Core Data, files |
| IPC | Intents, content providers, services | URL schemes, universal links, extensions |
| Test device | Emulator or rooted device | Simulator (limited) or jailbroken device |

Static analysis (decompiling the package) and dynamic analysis (running the app instrumented) are
both standard; `MobSF` (Mobile Security Framework) automates a lot of the first pass for both
platforms.

## Characteristic weaknesses

### Insecure data storage

The most common mobile finding: sensitive data stored unprotected on the device — credentials,
tokens, PII in plaintext SQLite/preferences, secrets in logs, data left in caches or backups.
Because the attacker can read the device's storage, anything stored insecurely is exposed. The fix:
store secrets in the platform **Keystore (Android) / Keychain (iOS)**, encrypt sensitive local data,
and keep secrets off logs and out of backups.

### Insecure communication & certificate pinning

Apps talk to backends over the network, so you intercept that traffic with a proxy (Level 2 lesson 2)
and the device trusting your proxy CA. The twist: **certificate pinning** — the app hardwires the
server's expected certificate and refuses your proxy's cert, blocking interception. Testing a pinned
app requires **bypassing the pin** on a device you control (e.g. runtime hooking with **Frida** /
**Objection**). This is legitimate on your own/authorized app; it's how you see the traffic to test
the API. Findings: cleartext traffic, weak TLS, no pinning where it matters, or trusting
user-installed CAs.

### Hardcoded secrets

API keys, credentials and endpoints compiled into the app. Because the package can be decompiled, any
embedded secret is recoverable — a reason secrets belong on the backend, not in the client (the same
trust-boundary lesson again).

### Platform misuse & IPC

Exported Android components that shouldn't be, insecure URL-scheme handling on iOS, insecure
WebViews (JavaScript bridges exposing native functions), weak biometric/auth implementation.

### Backend (the big one)

All of Level 2 applies to the API the app calls: BOLA, broken auth, injection, mass assignment. Often
the richest findings are in the backend, reached by intercepting the app's traffic.

## Tooling

- **MobSF** — automated static + dynamic analysis for APK/IPA.
- **Frida / Objection** — runtime instrumentation: hook functions, bypass pinning, inspect/modify
  behaviour live.
- **jadx / apktool** (Android), **class-dump / Hopper** (iOS) — decompiling and inspecting.
- **Burp/ZAP** — intercept the backend traffic (once pinning is handled).
- Emulators/simulators, or rooted/jailbroken test devices you own.

## A worked example (reasoning)

Testing your own Android app: MobSF's static pass flags a hardcoded API key and a world-readable
SharedPreferences file. You install on a rooted emulator, log in, and inspect storage — the session
token is stored in plaintext (insecure storage finding). You try to proxy the backend traffic; the
app uses certificate pinning, so you use Objection/Frida to bypass the pin on your device, then
intercept the API calls and test them for BOLA and broken auth (Level 2 lessons 6, 9). You document:
the hardcoded key (recoverable by decompiling — must move server-side), the plaintext token (must use
the Keystore), and any backend API flaws — each with evidence and fix, all against your own app.

## How It Actually Works

Why is "insecure local storage" and "hardcoded secret" such a persistent mobile problem, when the web
largely doesn't have the first and treats the second as obviously wrong? Because mobile development
*feels* like you control the client in a way web never pretended. A web front end is plainly running
in someone's browser; a mobile app feels like software you built and shipped, running on a phone,
and that illusion of control tempts developers to treat the device as a safe place to keep things —
a token here, an API key there, some logic that "the user won't see." But the device is hostile
hardware: it can be rooted or jailbroken, the package can be decompiled back to near-source, storage
can be read byte for byte, and running functions can be hooked and rewritten at runtime with Frida.
So the trust boundary from Level 2 lesson 1 applies *even harder* — not only is client-sent data
untrusted, but *nothing shipped in the client is secret*. A hardcoded key isn't "hidden in the
binary"; it's published to everyone who downloads the app, just slightly encoded. A token in
SharedPreferences isn't "stored on the user's device"; it's stored where any attacker with the device
can read it. The mechanism behind every one of these findings is the same realisation: the client is
the attacker's territory, so secrets and trust must live on the backend.

Certificate pinning is the interesting inversion — here the app is trying to *enforce* the trust
boundary in the app's favour, refusing to talk to anyone but the real server, and your test has to
defeat the app's own defence to proceed. That's not a contradiction: pinning protects the app's
users against network attackers (a real man-in-the-middle), which is good; but *you*, testing your
own app on your own device, need to see the traffic to assess the backend, so you bypass the pin
locally with runtime instrumentation — exactly the "I consent to intercept my own traffic" move from
Level 2 lesson 2, now applied to a native app. The deeper point is that pinning, like all client-side
controls, raises the attacker's effort (they must bypass it on a controlled device) but cannot be
relied on as the *only* defence, because an attacker who controls the device can always bypass it —
which is precisely why the backend must still enforce authentication and authorization itself.
Mobile security, in the end, is web security's trust boundary taken to its logical extreme: assume
the client is fully compromised, and put every real control on the server.

## Common mistakes and pitfalls

- **Testing only the app, not the backend.** The richest findings are often in the API the app
  calls. Intercept and test it as a web/API target (Level 2).
- **Giving up at certificate pinning.** Pinning blocks a naive proxy, not a determined tester on a
  controlled device; bypass it with Frida/Objection to test the backend.
- **Trusting that a hardcoded secret is "hidden."** Decompilation recovers it. Any client-shipped
  secret is public; move it server-side.
- **Ignoring local storage.** Plaintext tokens/PII in preferences, SQLite, logs and backups are the
  most common mobile finding.
- **Testing on a device you don't control / an app you're not authorized for.** Use emulators or your
  own rooted/jailbroken devices and authorized apps only.
- **Assuming client-side controls (pinning, root detection, obfuscation) are security.** They raise
  effort; the real controls must be server-side.

## Exercise

1. Describe the mobile attack surface in four parts (on-device app, local storage, network traffic,
   backend) and give one characteristic weakness for each.
2. Run MobSF against an APK/IPA you own. List the top three issues it flags and classify each
   (storage, secret, communication, platform).
3. Explain why a hardcoded API key in a mobile app is effectively public, referencing decompilation.
4. Describe how you'd test a pinned app's backend traffic on a device you control, and why bypassing
   the pin locally is legitimate for your own app.
5. In your own words, explain how mobile is "the web trust boundary taken to its extreme," and why
   that means every real control belongs on the backend.
