# Learning path — 2-layer to a 4-layer RF board

For someone who is comfortable with 2-layer boards in KiCad and has never done 4 layers.
Everything below is a verified link. Total core viewing: **~9 hours.**

---

## 1. KiCad or Altium — the honest answer

**Stay in KiCad for this board. Learn Altium after it flies.**

Altium is genuinely a better tool. Its interactive router, constraint manager, multi-channel design and library management are all ahead of KiCad, and it is what most hardware companies in aerospace actually use — so learning it is a good career investment.

None of that helps you here.

| | KiCad 9 | Altium |
| --- | --- | --- |
| Does everything this board needs? | **Yes** — 4-layer stackup, impedance calculator, custom DRC rules, JLCPCB export plugin | Yes |
| You already know it | **Yes** | No |
| Learning curve while also learning 4-layer + RF | — | Two new things at once |
| Cost | Free | Free student licence, otherwise ~$4k/yr |
| JLCPCB output path | Well-trodden, one-click plugin | Works, more manual |

**The real argument:** your first 4-layer board with an RF trace already introduces stackup, impedance, plane management and return-current reasoning — four new concepts. Adding "and I am also learning where the buttons are" doubles the number of things that can go wrong, on a project with a shipping round-trip to Uzbekistan in the critical path.

The concepts are what matter, and they transfer completely. Phil's Lab teaches the same material in both tools, so you can watch an Altium video and apply it in KiCad without difficulty.

**When you do want Altium:**

- [Altium Student Lab — free licence](https://www.altium.com/education/students) — free for enrolled students with a university email
- [Altium Education FAQs](https://education.altium.com/p/faqs) — eligibility and renewal
- [Fedevel free Altium tutorial for beginners](https://fedevel.com/courses/free-altium-tutorial-for-beginners) — Robert Feranec's free course
- [Phil's Lab #41 — STM32 PCB in Altium + JLCPCB](https://www.youtube.com/watch?v=PMEpQZ90f34) — the same board type you are building, in Altium

Do this **after** the WIT board is ordered, as a second pass on a simple project.

---

## 2. What actually changes going from 2 to 4 layers

Short list, because it is shorter than people expect:

1. **You now have a reference plane.** Every signal has continuous ground beneath it. This is the entire benefit and the entire responsibility.
2. **Impedance becomes a real, calculable number.** Trace width plus dielectric height gives you a defined impedance. On 2 layers you were ignoring this and mostly getting away with it.
3. **You must think about which plane a signal references**, and never let a trace cross a gap in that plane.
4. **Power becomes polygons on L3, not traces.**
5. **Via count goes up** — every layer change needs one, and every ground pin wants its own.

That is it. Routing itself is *easier* than 2-layer, not harder, because you stop weaving around traces.

**The one habit to unlearn:** on 2-layer boards you probably poured ground on the bottom and treated it as "good enough." On 4 layers the plane is sacred. Cut it once and you have undone the reason you paid for four layers.

---

## 3. Watch in this order

### Stage A — 4-layer mechanics (1 hour)

| # | Video | Time | Why |
| --- | --- | --- | --- |
| 1 | [EEVblog #1193 — KiCad PCB 4-Layer Swapping & Stackup](https://www.youtube.com/watch?v=2v5IvaWrPKk) | ~25 min | The practical mechanics of setting up and changing a 4-layer stackup in KiCad. Exactly the gap you have. |
| 2 | [KiCad Tutorial — How to make a 4-layer PCB](https://www.youtube.com/watch?v=RlB2mzoIKbo) | ~20 min | Second pass on the same material from a different angle. Worth it — this is the one thing you have never done. |
| 3 | [JLCPCB — PCB stackup best practices](https://jlcpcb.com/blog/pcb-stackup-best-practices) | 15 min read | From the fab you are actually ordering from. Read it before you pick a stackup. |

### Stage B — grounding and return current (2.5 hours) ⭐ THE MOST IMPORTANT ONE

| # | Video | Time | Why |
| --- | --- | --- | --- |
| 4 | [Rick Hartley — How to Achieve Proper Grounding](https://www.youtube.com/watch?v=ySuUZEjARPY) | ~2 h 20 min | **Watch this one in full. Twice if you can.** It is the single best explanation of return current, why split ground planes are harmful, and why "ground" is not a place electrons go to die. It will change how you lay out every board for the rest of your career. Also on the [Altium resources page](https://resources.altium.com/p/us-how-to-achieve-proper-grounding-rick-hartley-expert-live-training). |

If you watch **one** thing on this list, make it this. Everything in the grounding section of `pcb/stackup_and_layout.md` is downstream of this lecture.

### Stage C — the board you are actually building (3 hours)

| # | Video | Time | Why |
| --- | --- | --- | --- |
| 5 | [Phil's Lab #15 — KiCad STM32 Hardware Design in 20 Minutes](https://www.youtube.com/watch?v=wLwKgMBWhpY) | 20 min | Fast orientation. Watch first. |
| 6 | [Phil's Lab #65 — KiCad STM32 PCB Design Full Tutorial](https://www.youtube.com/watch?v=aVUqaB0IMh4) | ~2.5 h | **The single most relevant video in existence for this project.** A complete STM32 board in KiCad, start to finish, including stackup, decoupling, routing and fab output. Follow along and build his board before you build yours. |
| 7 | [Phil's Lab #11 — KiCad STM32 + USB + Buck Converter + JLCPCB assembly](https://fedevel.com/blog/kicad-stm32-usb-buck-converter-pcb-design-and-jlcpcb-assembly-update-phils-lab-11) | ~1 h | Adds the switching regulator and the JLCPCB assembly workflow. Both are in your design. |

### Stage D — impedance and RF (2 hours)

| # | Video | Time | Why |
| --- | --- | --- | --- |
| 8 | [Phil's Lab #3 — KiCad Controlled Impedance Traces (50 Ω)](https://www.youtube.com/watch?v=0fteCxn5XXA) | ~20 min | Exactly the calculation you need for the one RF trace, in your tool. |
| 9 | [Phil's Lab #171 — PCB Controlled Impedance](https://www.youtube.com/watch?v=aNYmOIMt7CY) | ~30 min | The deeper version. Why the number is what it is. |
| 10 | [Phil's Lab #19 — Practical RF Hardware and PCB Design Tips](https://www.youtube.com/watch?v=_Hfzq1QES-Q) | ~30 min | Keep-outs, stitching vias, component placement near RF. Directly applicable. |
| 11 | [Phil's Lab #139 — PCB Chip Antenna Hardware Design](https://www.youtube.com/watch?v=UQBMROv7Dy4) | ~30 min | You are using connectorised antennas, not chip antennas, but the ground-plane and clearance reasoning is the same. |
| 12 | [JLCPCB — 50 Ω trace with simple Pi matching](https://jlcpcb.com/blog/pcb-basics-50ohm-pcb-trace-with-simple-pI-matching) | 10 min read | Your fab's own guidance. |
| 13 | [Tech Explorations — KiCad impedance and stackup calculations](https://techexplorations.com/guides/kicad/high-speed-pcb-design/layout-impedance-and-stack-up-calculations/) | 15 min read | Written reference for the KiCad calculator. |

### Stage E — power layout (45 min)

| # | Resource | Why |
| --- | --- | --- |
| 14 | [Phil's Lab / Altium — Switching Buck Converter Component Sizing](https://resources.altium.com/p/buck-converter-component-sizing) | How to pick the inductor and capacitors from the datasheet rather than copying blindly. You have a buck-boost to size. |
| 15 | The **layout section of the TPS63020 datasheet** | Every TI regulator datasheet has a layout section with a recommended placement diagram. Copy it exactly. This is the highest-value 10 minutes in the power design. |

---

## 4. Channels worth subscribing to

| Channel | What it is | Use for |
| --- | --- | --- |
| [**Phil's Lab**](https://www.youtube.com/@PhilsLab) | Phil Salmony — STM32, KiCad, Altium, RF, DSP. Aerospace background. | **Your primary resource.** Closest match to this project of anything on YouTube. [Video index](https://www.phils-lab.net/videos) |
| [**Robert Feranec**](https://www.youtube.com/c/RobertFeranec) | FEDEVEL — motherboard-class design, Altium, interviews with industry engineers | Deeper hardware design process, and your Altium on-ramp later |
| [**Shawn Hymel / DigiKey**](https://www.youtube.com/playlist?list=PLEBQazB0HUySNug4eRm-73hNyMcCRViRB) | Structured KiCad introduction ([Part 1](https://www.youtube.com/watch?v=vaCVh2SAZY4)) | Filling gaps in KiCad fundamentals |
| [**Eric Bogatin**](https://colorado.edu/faculty/bogatin/youtube-videos) | University of Colorado — signal and power integrity, from first principles | When you want to understand *why*, not just *what* |

There is also a [community-compiled Phil's Lab learning path](https://gist.github.com/JonyBepary/ed7419fec1ed115fd9aed64be34afde5) that sequences his videos beginner-to-advanced.

---

## 5. Books, if you want depth

| Book | Author | Worth it because |
| --- | --- | --- |
| *Electromagnetic Compatibility Engineering* | Henry Ott | The authoritative text on grounding and PCB layout. The grounding chapters alone justify it. |
| *Signal and Power Integrity — Simplified* | Eric Bogatin | The standard reference. "Simplified" is relative, but it is readable and every concept has a physical explanation. |
| *High-Speed Digital Design: A Handbook of Black Magic* | Johnson & Graham | Older, still excellent on transmission lines and return paths. |

You do not need any of these to build this board. Buy Ott if you intend to keep doing hardware.

---

## 6. Do this before you lay out the real board

**Build a practice board first.** Not optional — this is how you convert watching into knowing.

1. **Follow Phil's Lab #65 exactly**, building his board alongside the video in KiCad. Do not skip ahead or "just watch." 2–3 evenings.
2. **Then do a throwaway 4-layer exercise of your own**: an STM32F407 with two sensors, a buck-boost, and one U.FL connector with a 50 Ω trace. 60 × 60 mm. Do not order it. The point is to make every mistake once, cheaply.
3. **Check your practice board against §4 of `pcb/stackup_and_layout.md`** — display L2 alone, display the RF net alone. See whether your instincts produced an unbroken plane.
4. **Only then start the WIT layout.**

This costs you roughly a week and it is the cheapest week in the project. A respin costs three.

---

## 7. What not to spend time on

Things that look important and are not, for this board:

- **Differential pair length matching.** Your fastest signal is USB full-speed at 12 Mbps. Length matching matters above ~100 Mbps. Route them together, keep them reasonable, move on.
- **Via stitching everywhere at calculated spacing.** Along the RF trace it matters. Everywhere else, every 5–10 mm by eye is fine.
- **Simulation.** No SPICE, no field solver, no EM simulation. You have one short RF trace and a module with a certified front end. The KiCad calculator is sufficient.
- **Impedance-controlled fabrication.** Costs money and lead time. Not needed for a 15 mm trace — see the stackup document.
- **Learning Altium now.** Covered above.
- **Teardrops, fancy via-in-pad, back-drilling.** None of it applies at these speeds.

The failure modes on this board are **mechanical and procedural** — a wrong footprint, a backwards connector, a broken ground plane, a CPL rotation error. Not exotic signal integrity. Spend your attention accordingly.

---

## 8. A realistic schedule for the learning

| Week | Activity | Hours |
| --- | --- | --- |
| Week 1 | Stage A + Stage B (Rick Hartley in full) | ~4 |
| Week 1–2 | Stage C — follow Phil's Lab #65 and build his board | ~8 |
| Week 2 | Stage D + E | ~3 |
| Week 2–3 | Your own throwaway 4-layer practice board | ~10 |
| Week 3+ | WIT schematic and layout | — |

**Run this in parallel with firmware on the dev board and with procurement.** None of the three block each other, and the long-lead components should already be on their way while you are watching videos.
