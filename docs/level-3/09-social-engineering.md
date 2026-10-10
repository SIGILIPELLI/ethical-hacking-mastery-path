---
description: "Social engineering and authorized phishing simulation: the psychology attackers exploit, phishing and pretexting, and running a safe simulation."
---

# Social Engineering & Authorized Phishing Simulation

Technical controls can be excellent and the organisation still falls — because the easiest path in is
often a *person*, not a system. Social engineering manipulates people into revealing information,
granting access, or performing actions that undermine security. It is a legitimate part of many
engagements, but it is also the area with the sharpest ethical and legal constraints, because it
targets *humans*. This lesson covers the techniques, the psychology, and — most importantly — how to
run an **authorized, ethical phishing simulation** without harming the people involved.

!!! danger "People are the target — the rules are stricter"
    Social engineering is only ever performed with explicit written authorization that *specifically*
    covers it (it's usually a separate line item in the scope, not implied by a network pentest). You
    never collect real passwords beyond proof, never shame individuals, never use genuinely harmful
    pretexts (fake emergencies about family, real financial threats), and you handle all results with
    extreme care. Run simulations only in authorized engagements or against willing participants.

## Why social engineering works: the psychology

Attackers exploit reliable human tendencies — the same ones that make us functional, cooperative
people:

- **Authority** — we comply with apparent authority ("This is IT, we need your password to fix your
  account").
- **Urgency / scarcity** — pressure short-circuits careful thinking ("Your account will be locked in
  1 hour").
- **Trust / familiarity** — a message that looks like it's from a colleague, a known vendor, or a
  real brand.
- **Reciprocity** — a small favour creates a sense of obligation.
- **Social proof** — "everyone on your team has already done this."
- **Helpfulness / fear** — most people want to help, and fear of consequences drives compliance.

Good awareness training teaches people to recognise these triggers; good phishing simulations measure
whether that training stuck.

## The techniques

- **Phishing** — fraudulent emails at scale, luring recipients to a fake login page or a malicious
  attachment.
- **Spear phishing** — targeted phishing using specifics about the person (from OSINT, Level 1 lesson
  6) to be convincing.
- **Whaling** — spear phishing executives.
- **Vishing** — voice/phone pretexting ("Hi, this is the help desk…").
- **Smishing** — phishing over SMS.
- **Pretexting** — an invented scenario/identity to extract information or access.
- **Baiting** — leaving "lost" USB drives or offering something enticing.
- **Tailgating / piggybacking** — physically following someone through a secure door.

## Running an authorized phishing simulation

A professional simulation is a careful, consented exercise — not a trick to embarrass staff. The
workflow:

1. **Authorization & objectives.** Written scope that explicitly includes social engineering, agreed
   with leadership. Define the goal: measure click rates? credential submission? reporting rates?
   Usually leadership consents on behalf of the org; individuals are not told in advance (or the test
   is meaningless) but the *organisation* has consented and agreed on handling.
2. **Design the pretext.** Realistic but **not harmful** — a plausible internal notice or vendor
   email. Avoid themes that cause real distress (layoffs, bonuses, family emergencies, real legal
   threats). Ethics boards and good firms explicitly forbid cruel pretexts.
3. **Build the infrastructure (lab/engagement).** A tool like **GoPhish** (open source) sends the
   campaign and hosts a **tracking** landing page. The landing page should capture *that a click or
   submission happened* — **not** store real passwords. Best practice: the "login" page records the
   attempt and redirects to a training message, discarding any entered credential.
4. **Run it** within the agreed window.
5. **Measure** — click rate, submission rate, and crucially the **reporting rate** (how many people
   reported the phish to security). Reporting rate is the metric that actually reflects resilience.
6. **Debrief and train, not blame.** Deliver results as aggregate statistics; turn clicks into a
   teachable moment with immediate training; never single out or punish individuals. The goal is a
   stronger organisation, not a list of "failures."

```text
# Conceptual GoPhish flow (run only in an authorized engagement):
#   sender profile -> email template (safe pretext) -> landing page (records click,
#   does NOT store passwords, redirects to training) -> campaign to the agreed group
#   -> dashboard of click/submit/report rates (aggregate)
```

## A worked example (reasoning)

Scope authorizes a phishing simulation of 200 staff to measure susceptibility and reporting. You
design a benign "IT: please re-verify your mailbox" email, stand up a GoPhish campaign with a landing
page that logs clicks and shows training (never stores the typed password), and run it over a week.
Results (aggregate): a portion clicked, fewer submitted, and some reported it to security. You report
the *rates*, the pretext, and recommendations — targeted training, easier phishing-report tooling,
and technical controls (DMARC, link rewriting, MFA so a phished password alone fails). No individual
is named; the deliverable strengthens the human layer.

## The defences

- **Regular, non-punitive awareness training** and ongoing simulations that trend the metrics over
  time.
- **Easy reporting** — a one-click "report phish" button; reward reporting, don't punish clicking.
- **Technical backstops** so a human mistake isn't fatal: **MFA** (a phished password alone fails),
  email authentication (**SPF/DKIM/DMARC**), link/attachment sandboxing, external-sender banners.
- **Process controls** — verify unusual requests (especially financial) via a second channel;
  callback procedures for the help desk; visitor/physical-access policies for tailgating.
- **A culture where it's safe to say "I clicked something"** — fear of blame delays reporting, which
  is the worst outcome.

## How It Actually Works

Why is social engineering so effective against organisations with strong technical security, and why
does measuring the *reporting* rate matter more than the *click* rate? Because the techniques don't
attack a flaw in a system — they exploit features of human cognition that are adaptive almost all the
time. Deferring to authority, acting on urgency, trusting the familiar, wanting to help: these make
humans cooperative and efficient, and they can't be "patched" out the way a SUID bug can, because
the same instincts are what make a workforce function. An attacker only needs the instinct to win
*once*; the defender needs it to not fire at the wrong moment across thousands of people and millions
of messages. That asymmetry is why no amount of technical control fully closes the human path, and
why the realistic goal is *resilience*, not *immunity* — reduce the rate, add technical backstops so
a single lapse isn't catastrophic, and shorten the time between "someone clicked" and "security
knows."

That last point is why reporting rate is the key metric. Click rate measures susceptibility, but
clicks will never be zero — someone always will. What determines whether a phish becomes a breach is
*how fast the organisation reacts*: a phish reported in two minutes lets security pull the email,
reset the account and block the domain before the attacker pivots; a phish nobody reports gives the
attacker hours. A culture that punishes clicking drives the opposite of what you want — people hide
their mistakes, reporting collapses, and reaction time balloons. So the ethical design of the
simulation (aggregate stats, training not blame, rewarding reports) isn't just politeness; it's what
actually produces the behaviour that reduces real risk. The whole discipline rests on treating people
as part of the defence to be strengthened, not as a failure to be caught — which is also exactly why
cruel pretexts and individual shaming are forbidden: they damage the very thing you're trying to
build.

## Common mistakes and pitfalls

- **Doing any social engineering without explicit, specific authorization.** It targets people; the
  consent bar is higher than for network testing, and it's usually a separate scope item.
- **Storing real passwords from a simulation.** Never. Record that a submission happened and discard
  the credential; a captured-password store is a liability and an ethics breach.
- **Harmful pretexts.** Fake family emergencies, real bonus/layoff themes, genuine legal threats —
  off-limits. Keep pretexts plausible but harmless.
- **Blaming individuals / publishing names.** Destroys reporting culture and trust. Report aggregates;
  train, don't shame.
- **Measuring only click rate.** The reporting rate is the metric that predicts real-world resilience.
- **Treating the simulation as the deliverable.** The deliverable is improved resilience — training,
  tooling, and technical backstops (MFA, DMARC) that make the human layer forgiving.

## Exercise

1. List the six psychological triggers from this lesson and, for each, write a one-line example of a
   phishing message that exploits it.
2. Draft a *safe, ethical* phishing pretext suitable for an authorized simulation. Then write one
   genuinely harmful pretext and explain why it's off-limits.
3. Describe how a simulation landing page should handle a submitted password, and why it must not
   store it.
4. Explain why the reporting rate is a more important metric than the click rate, using the
   reaction-time argument.
5. For an organisation, list three technical backstops that limit the damage when (not if) a user is
   phished, and say what each one neutralises.
