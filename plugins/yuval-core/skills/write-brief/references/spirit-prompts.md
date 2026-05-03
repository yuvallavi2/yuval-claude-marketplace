# Spirit prompts

The 8 prompts walked when authoring or refreshing a project's `/code/SPIRIT.md`. Each prompt is one line; each example is one line. `n/a` is a valid answer for any prompt — pure-infrastructure projects with no user surface answer `n/a` to all 8.

Used by both `write-brief` (on first invocation in a project — when `/code/SPIRIT.md` is absent) and `refresh-spirit` (on every invocation — pre-filled with current SPIRIT.md content). Per D-016 each skill carries its own copy; cross-skill drift is tolerated, not policed.

---

## 1. Reference Scenario

**Q:** Narrate one end-to-end first-person moment of the product being used. What is the user doing, saying, hearing? What is their body doing? Include time of day, voice/text quotes, and the texture of the moment.

**Example:** *"It's 7am. I'm lacing my running shoes. I tap my AirPod twice and say 'morning'. Mira replies in my ear: 'Two things on your plate today — the board update at 11 and Dana's onboarding at 4. Want me to draft the board update while you run?' I say 'yeah, focus on the customer numbers.' Eyes never leave the door."*

---

## 2. Interaction Shape

**Q:** What is the primary surface? Pick one: `conversation | form | list | canvas | map | timeline | dashboard | mixed`. If `mixed`, rank and name the dominant.

**Example:** *"Conversation (voice-primary, text-secondary). No form. No dashboard."*

---

## 3. Voice / Tone samples

**Q:** Give 3–5 concrete script lines the product would say, with the moment each fires. Testable, not vibes.

**Example:**
- *On wake: "Morning. Two things on your plate."* (concise, declarative, no greeting fluff)
- *On surprise: "Hm, that's new — give me a second."* (admits uncertainty, doesn't fake confidence)
- *On error: "I lost the thread. Say it again?"* (own the failure, no blame-shift)
- *On goodbye: silence.* (no "have a great day")

---

## 4. Anti-Shape

**Q:** What UI/UX patterns must this product NOT become? Name them explicitly — negative framing kills drift toward the familiar wrong shape.

**Example:** *"Not a chatbot window. Not a form with dropdowns. Not a notification center. Not a 'tap here to start' modal. Not a productivity dashboard with streaks and badges."*

---

## 5. Silent State

**Q:** What does the product look like at rest? When the user isn't actively using it, what is it doing or showing?

**Example:** *"Invisible. No always-on widget. No menu-bar icon pulsing. No badge. Activated only by the wake phrase or a single-tap gesture; otherwise dormant."*

---

## 6. Tactile Contract

**Q:** What is the user's body doing while they use this? Voice / keyboard / mouse / touch / passive. Hands-busy or hands-on. Eyes on/off screen.

**Example:** *"Hands busy (running, cooking, driving). Eyes off-screen by default. Voice in, voice out. Optional single-tap on AirPod for wake; no other touch interaction."*

---

## 7. Default Posture

**Q:** What is the product's default stance? One line. Choices like: confrontational by default, silent by default, async by default, agreeable by default, deferential by default. Each propagates across hundreds of UI decisions.

**Example:** *"Silent by default. Speaks only when spoken to or when something genuinely time-sensitive surfaces. No proactive nudges, no streaks, no 'you haven't checked in today' nonsense."*

---

## 8. The One Moment

**Q:** If everything else were wrong but one moment right, would it still be the product? Name that moment.

**Example:** *"The morning briefing while lacing running shoes. If that one moment lands — concise, hands-free, no friction — the product works even if the rest is rough. If that moment misses, nothing else matters."*

---

## n/a — pure infrastructure projects

If the project has no user-facing surface (a framework, a CLI tool, a library, a build system), all 8 prompts may be answered `n/a`. In that case, write the entire SPIRIT.md as a single line:

```
n/a — pure infrastructure project. No user surface, no atmosphere to transmit.
```

Acceptance criteria fully cover intent for this kind of project. Don't fabricate atmosphere where there isn't any.
