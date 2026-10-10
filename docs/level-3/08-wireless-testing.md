---
description: "Wireless network testing: how Wi-Fi security works, WPA2 vs WPA3, the handshake capture-and-crack attack, evil twins, and testing your own AP."
---

# Wireless Network Testing

Wireless adds a physical dimension no firewall covers: the signal leaves the building. An attacker in
range doesn't need to be plugged into anything. This lesson covers how Wi-Fi security works, the
characteristic attacks (capturing a handshake and cracking it, rogue access points), and the hard
legal boundary that makes wireless testing different: **radio is shared, so you test only your own
access point.**

!!! danger "Wireless has an extra-bright legal line"
    Capturing or interfering with Wi-Fi you don't own affects everyone in range and is explicitly
    illegal in most jurisdictions (interception and unauthorized-access statutes). On a real
    engagement, wireless testing needs specific written authorization and a defined physical area.
    For learning, test **only your own access point** — set one up for the purpose. This course
    describes the techniques; it does not run them against any network.

## How Wi-Fi security works

- **Open networks** — no encryption; anyone in range reads the traffic. (Captive portals don't
  encrypt the air.)
- **WEP** — the original, cryptographically broken decades ago; crackable in minutes. If you find it,
  the finding is "WEP is in use" — it provides no meaningful protection.
- **WPA2-PSK (Pre-Shared Key)** — the long-standard home/small-business scheme. A single passphrase;
  security rests on its strength and on the 4-way handshake.
- **WPA2-Enterprise (802.1X)** — per-user authentication via a RADIUS server (certificates or
  credentials), no shared passphrase. Stronger, but introduces credential-relay and rogue-AP risks.
- **WPA3** — the current generation; replaces the PSK handshake with **SAE (Simultaneous
  Authentication of Equals)**, which is designed to resist the offline cracking that WPA2-PSK allows.

## Requirements (hardware reality)

Wireless testing needs a Wi-Fi adapter that supports **monitor mode** (to capture raw frames) and
**packet injection** (to send frames, e.g. deauthentication). Not all adapters do; many built-in
laptop/VM adapters can't, which is why testers buy specific USB adapters with supported chipsets.
This is a genuine hardware dependency — **this course cannot run live wireless captures**, and does
not print capture output it didn't produce. The toolset is the **aircrack-ng** suite
(`airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`), plus `hcxdumptool`/`hashcat` for modern
workflows.

## The characteristic WPA2-PSK attack: capture and crack

WPA2-PSK's weak point is **offline**: you don't attack the live network, you capture one piece of a
legitimate connection and crack it at leisure.

1. **Monitor mode** — put the adapter into monitor mode to see raw 802.11 frames
   (`airmon-ng start wlan0`).
2. **Find your AP** — `airodump-ng` lists networks, channels and connected clients. Target *your
   own* AP's BSSID and channel.
3. **Capture the 4-way handshake** — when a client joins, the AP and client exchange a 4-message
   handshake that contains a value derived from the passphrase. `airodump-ng` captures it.
4. **(Optionally) force a reconnection** — a **deauthentication** frame kicks a client off so it
   reconnects, letting you capture the handshake sooner. (Deauth affects real users — your own
   devices only.)
5. **Crack offline** — feed the captured handshake and a wordlist to `aircrack-ng`/`hashcat`. The
   crack succeeds only if the passphrase is in your wordlist. A long, random passphrase won't be.

The finding this produces is almost always about **passphrase strength**: a weak WPA2-PSK passphrase
is crackable from a single captured handshake; a strong one is not.

### PMKID attack

A refinement: some APs leak a **PMKID** in a single frame, letting you capture the crackable material
*without* waiting for a client to connect (`hcxdumptool` → `hashcat`). Same offline-crack principle,
fewer prerequisites.

## Rogue APs and evil twins

- **Evil twin** — a fake AP broadcasting a legitimate network's name (SSID) to lure clients into
  connecting, so you can capture credentials or man-in-the-middle traffic. Effective against open and
  enterprise networks and against users who auto-connect to known SSIDs.
- **Rogue AP** — an unauthorized AP plugged into a corporate network, bridging the air to the wired
  LAN and bypassing perimeter controls.
- **Karma-style attacks** — responding to devices' probe requests for remembered networks.

These target the *client's* trust in a network name, which is why enterprise clients should validate
the RADIUS server's certificate (so they won't hand credentials to an evil twin) — and often don't.

## The defences

- **Use WPA3** where supported; otherwise **WPA2 with a long, random passphrase** (length defeats the
  offline crack). Never WEP or open for anything sensitive.
- **WPA2/WPA3-Enterprise** for organisations, with **server-certificate validation enforced on
  clients** (so they won't trust an evil twin).
- **Wireless intrusion detection (WIDS)** to spot rogue APs, deauth floods and evil twins.
- **Network segmentation** — treat Wi-Fi as untrusted; put it on its own segment with controlled
  access to internal resources.
- **Disable WPS** (its PIN has its own well-known weaknesses).

## How It Actually Works

Why can WPA2-PSK be attacked entirely *offline*, and why does WPA3 largely fix it? WPA2-PSK derives
the keys that encrypt your session from the shared passphrase plus public values exchanged in the
4-way handshake (the SSID, nonces, MAC addresses). Crucially, everything needed to *test a passphrase
guess* is either public or captured in that handshake: given a candidate passphrase, an attacker can
recompute the same derivation and check whether it produces a value matching the captured handshake.
So the handshake is effectively a verifier for guesses — capture it once, and you can try unlimited
passphrases offline, at full hardware speed, with the network none the wiser. This is the same
economics as password cracking (lesson 3): the defence isn't a cleverer protocol detail, it's
passphrase *entropy* — a passphrase not in any wordlist and long enough to be infeasible to brute
makes the offline attack useless even though the attacker holds a perfect verifier.

WPA3's SAE handshake changes the game by making the exchange a **password-authenticated key
agreement** (a dragonfly/PAKE-style protocol). Instead of the handshake yielding something an
attacker can test guesses against offline, SAE is constructed so that each guess requires a *fresh
interaction with the AP* — you can't take one capture and grind it offline; you'd have to make a
separate online attempt per guess, which is slow and detectable, exactly like an online login attack
with rate limiting. SAE also provides *forward secrecy*, so capturing traffic and later learning the
passphrase still doesn't decrypt past sessions. That's why the industry guidance is simply "use
WPA3": it converts the cheap, undetectable offline attack back into an expensive, detectable online
one. The rogue-AP/evil-twin attacks, by contrast, aren't about the crypto at all — they exploit that
a network's *name* (SSID) is unauthenticated and clients will trust a familiar name. The crypto can
be perfect and a client that connects to an attacker's look-alike AP and hands over credentials has
still lost — which is why certificate validation on the *client* side (proving the AP/RADIUS server
is the real one) is the matching defence. Air is a shared, spoofable medium; wireless security is a
constant effort to re-introduce the authentication and confidentiality that a wire gave for free.

## Common mistakes and pitfalls

- **Testing any network but your own.** The legal line is brightest here. Set up your own AP;
  authorised engagements need explicit, area-defined permission.
- **Buying an adapter without checking monitor-mode/injection support.** Most built-in adapters
  can't do it; the whole workflow depends on a supported chipset.
- **Deauthing broadly.** Deauth frames knock *everyone* off; even on your own AP, be precise. In the
  wild it's disruption of service — an attack.
- **Expecting to crack a strong WPA2 passphrase.** A captured handshake only cracks if the passphrase
  is in your wordlist. Length/randomness defeats it entirely.
- **Assuming WPA2-Enterprise is immune.** Without client-side certificate validation, evil twins
  harvest enterprise credentials. The protocol is fine; the client config is the hole.
- **Printing or trusting "handshake captured" output you didn't actually produce.** Wireless needs
  real hardware; don't fabricate it.

## Exercise

1. Set up your *own* Wi-Fi access point for testing. Configure it once with a short dictionary-word
   passphrase and once with a long random one (you'll reason about both).
2. Describe, step by step, the capture-and-crack workflow against your own AP (monitor mode → find AP
   → capture handshake → crack). If you have supported hardware, run it against your own AP and record
   whether each passphrase cracked.
3. Explain why the attack is *offline* and what that means for passphrase-strength requirements.
4. Explain, in your own words, how WPA3's SAE handshake prevents the offline attack that WPA2-PSK
   allows.
5. Describe an evil-twin attack and the specific *client-side* configuration that defeats it, and
   explain why the defence lives on the client rather than in the crypto.
