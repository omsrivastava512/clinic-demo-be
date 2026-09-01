# Specialty Treatment UX — Approach Comparison

Plain ASCII wireframes. No color, no CSS, no JS — layout and states only,
so the comparison is about function, not polish. `[Tile]` = a tappable
item. `( Button )` = a distinct action, different in kind from a tile.
`╔═╗` = a modal, floating above the screen it was opened from.

This file stands on its own — read it without needing the rest of the
SPECIALTY chat open alongside it.

---

## State 0 — The problem, as it exists in the real screenshot today

```
┌────────────────────────────────────────────────────────┐
│ Log Today's Procedures for Amit Trivedi                │
│ Complaints: Chronic Lower Back Pain, Post-Op ACL Rehab │
├────────────────────────────────────────────────────────┤
│ ① Chronic Lower Back Pain (L4-L5)                      │
│   [Consultation] [Ultrasonic Therapy]                  │
│   [IFT]           [TENS]                               │
│   [Manual Therapy][Short Wave Diathermy]               │
│   [Cervical Traction][Kinesio Taping]                  │
│   ┌───────────────────────────────────┐                │
│   │ 🔍 Search less common procedures.. │ <- Cupping    │
│   └───────────────────────────────────┘    only HERE   │
│                                                        │
│ ② Post-Op ACL Rehab (Right Knee)                       │
│   [Consultation] [Ultrasonic Therapy] ...              │
└────────────────────────────────────────────────────────┘
```

**The problem:** Cupping is only reachable by searching *inside one
complaint's box*. Nothing lets you say "this also covers ②." A service
seeker with no complaint section at all has nowhere to even search from.

---

## Approach A — A third section, shaped like the first two (rejected)

```
┌────────────────────────────────────────────────────────┐
│ ① Chronic Lower Back Pain (L4-L5)                      │
│   [Consultation] [Ultrasonic Therapy] [IFT] ...        │
│                                                        │
│ ② Post-Op ACL Rehab (Right Knee)                       │
│   [Consultation] [Ultrasonic Therapy] ...              │
│                                                        │
│ ③ Add Specialty Treatment         <- SAME visual shape │
│   [Cupping] [Dry Needling] [Laser]    as ① and ②       │
│   ↳ tap "Cupping" →                                    │
│     [✓Neck] [✓Shoulder] [ Lower Back ] <- but picking  │
│                                           COMPLAINTS   │
│                                           under SERVICE│
│                                           backwards    │
│                                           from ① and ② │
└────────────────────────────────────────────────────────┘
```

**Why it's jarring, visually:** ① and ② are `[complaint header]` over
`[procedure list]` — fixed header, variable body. ③ looks identical at
a glance, but is actually `[service header]` over `[complaint list]` —
the fixed and variable roles have silently flipped. Nothing above signals
the reversal until you actually engage with section ③.

---

## Approach B — Split entry point (Om's original proposal, SPECIALTY-07)

```
DAILY LEDGER
┌────────────────────────────────────────────────────────┐
│ 🔍 Search patient name or phone...                     │
│                                                        │
│ Amit Trivedi (98765xxxxx)                              │
│   ( Log Visit )       ( Log Specialty Treatment )      │
└────────────────────────────────────────────────────────┘
          │                              │
          ▼                              ▼
  COMPLAINT SELECTOR              SPECIALTY TREATMENT
  (existing screen,               (new, lighter screen)
   unchanged)                     ┌─────────────────────────┐
          │                       │ Logging: Amit Trivedi   │
          ▼                       │ Service:  [Cupping ▾]   │
  PROCEDURE LOGGER                │ Body part / complaint:  │
  (existing screen,               │  [ ]Neck [ ]Shoulder... │
   unchanged — no                 │ ( Save )                │
   specialty items here)          └─────────────────────────┘
```

**The real cost:** an active patient who wants *both* ordinary therapy
and cupping today has to run through **both** flows separately — finish
"Log Visit," return to the Daily Ledger, search for the *same* patient
again, then start "Log Specialty Treatment" from scratch.

---

## Approach C — Always-visible inline section (my SPECIALTY-07 recommendation)

```
PROCEDURE LOGGER — collapsed
┌────────────────────────────────────────────────────────┐
│ ① Chronic Lower Back Pain (L4-L5)                      │
│   [Consultation] [Ultrasonic Therapy] [IFT] ...        │
│                                                        │
│ ② Post-Op ACL Rehab (Right Knee)                       │
│   [Consultation] [Ultrasonic Therapy] ...              │
│                                                        │
│ SPECIALTY TREATMENTS                                   │
│   [Cupping] [Dry Needling] [Laser Therapy]             │
└────────────────────────────────────────────────────────┘

                    ...tap [Cupping]...

PROCEDURE LOGGER — mid-selection
┌────────────────────────────────────────────────────────┐
│ SPECIALTY TREATMENTS                                   │
│   [Cupping ✓] [Dry Needling] [Laser Therapy]           │
│   ↳ Applies to:                                        │
│      [✓Back Pain (L4-L5)] [✓ACL Rehab (Right Knee)]    │
│      (defaulted to both selected complaints —          │
│       tap either to remove it)                         │
└────────────────────────────────────────────────────────┘

Same component, reached via Approach B's entry point, for a
service seeker who has no complaints at all:
      [ ]Spine [✓]Shoulder [ ]Knee [ ]Hip [ ]Elbow [ ]Ankle
```

**Trade-off:** zero taps to *discover* it — it's always sitting there —
but it permanently occupies a section of the screen even on the far
more common visit where no specialty treatment happens at all.

---

## Approach D — A button that opens a modal (Om's new proposal, this message)

```
PROCEDURE LOGGER
┌────────────────────────────────────────────────────────┐
│ ① Chronic Lower Back Pain (L4-L5)                      │
│   [Consultation] [Ultrasonic Therapy] [IFT] ...        │
│                                                        │
│ ② Post-Op ACL Rehab (Right Knee)                       │
│   [Consultation] [Ultrasonic Therapy] ...              │
├────────────────────────────────────────────────────────┤
│             ( + Add Specialty Treatment )              │
└────────────────────────────────────────────────────────┘
                          │ tap
                          ▼
        ╔═════════════════════════════════════╗
        ║  ADD SPECIALTY TREATMENT      ( ✕ ) ║
        ║  Patient: Amit Trivedi              ║
        ║  Service:  [Cupping ▾]              ║
        ║  Applies to:                        ║
        ║   [✓Back Pain] [✓ACL Rehab]         ║
        ║  ( Cancel )               ( Save )  ║
        ╚═════════════════════════════════════╝

The SAME modal, reached directly from the Daily Ledger for a
service seeker who was never in a Procedure Logger session at all:

        ╔═════════════════════════════════════╗
        ║  ADD SPECIALTY TREATMENT      ( ✕ ) ║
        ║  Patient: [new] Priya Sharma        ║
        ║  Service:  [Cupping ▾]              ║
        ║  Applies to (body region):          ║
        ║   [ ]Spine [✓]Shoulder [ ]Knee ...  ║
        ║  ( Cancel )               ( Save )  ║
        ╚═════════════════════════════════════╝
```

**Why this dodges the earlier "Applies To button" rejection.** The
button rejected in SPECIALTY-07 was embedded *inside* the tappable
procedure list — one tile among many, secretly behaving differently
from every other tile around it, which is what made it cognitively
jarring. `( + Add Specialty Treatment )` here is never mixed into that
list at all — it sits outside it, visually and positionally, the same
way `( Cancel )` and `( Create Invoice )` already coexist as distinct
controls without confusing anyone. A button reads as a different *kind*
of control than a tile; the earlier rejection was about disguising one
as the other, not about buttons existing at all.

---

## Side-by-side comparison

```
APPROACH A — Inline 3rd section (inverted)
  Screen space when idle:    Always visible, a full section
  Taps for the common case:  n/a — broken as originally shaped
  Paradigm consistency:      POOR — header/body roles invert mid-screen
  Reaches service seeker:    No — still needs a complaint section to sit in

APPROACH B — Split entry point
  Screen space when idle:    Zero (a separate screen entirely)
  Taps for the common case:  A full second flow, including re-searching
                              the same patient
  Paradigm consistency:      Good — each screen is internally consistent
  Reaches service seeker:    Yes — this is its whole purpose

APPROACH C — Inline always-visible section
  Screen space when idle:    One section, permanently visible
  Taps for the common case:  2 (tap the service, confirm the chips)
  Paradigm consistency:      Good, provided it reads as clearly separate
                              from sections ① and ②
  Reaches service seeker:    Yes, via a second entry point reusing the
                              same component

APPROACH D — Button opens a modal
  Screen space when idle:    One line (a single button)
  Taps for the common case:  3 (open the modal, pick service, pick chips)
  Paradigm consistency:      Best — a button is recognizably a different
                              control than a tappable tile, not a
                              disguised version of one
  Reaches service seeker:    Yes — the identical modal, reached straight
                              from the Daily Ledger too
```

---

## Recommendation

D over C — a genuine change from what I recommended last time, not a
minor tweak. The deciding factor: most visits don't include a specialty
treatment at all, and C spends permanent screen space on the exception
rather than the rule. D keeps the default screen exactly as lean as it
is today, costs one extra tap only when specialty logging is actually
needed, and is *more* cleanly reusable across both patient archetypes
than C was — it's a single self-contained modal that only needs
"patient" and, optionally, "today's selected complaints," rather than a
section that has to know it's embedded inside a specific screen's
layout. The one thing worth watching: keep the button's position fixed
and consistent every time, so it doesn't quietly become undiscoverable
just because it isn't sitting in the main tappable list.
