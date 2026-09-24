# PCB stackup and layout guide

Target: **KiCad 9**, **4-layer**, **JLCPCB**, board ~90 × 90 mm.

---

## 1. Layer count: four

| | 2-layer | **4-layer** | 6-layer |
| --- | --- | --- | --- |
| Continuous ground under RF | Impossible | **Yes** | Yes |
| 50 Ω microstrip width | ~2.9 mm — unroutable | **~0.33 mm** | ~0.2 mm |
| Return-current integrity | Broken by every bottom trace | **Continuous** | Continuous |
| Bare PCB, 5 off, 100×100 | ~$5 | **~$8** | ~$45 |

Two layers fails on **return current**, not on routing difficulty. At 868 MHz the current returning from the antenna flows in the copper directly beneath the signal trace. If that copper is missing because the bottom layer carries an I²C trace there, the return detours, the detour is a loop, and the loop is an antenna. You get radiated emissions and a link that works at 2 m and dies at 2 km.

Six layers buys routing room you do not need — one RF trace, nothing faster than USB full-speed, a dozen ICs.

**Four layers costs ~$3 more across the whole order.**

---

## 2. Stackup — order JLC04161H-7628

| Layer | Name | Copper | Dielectric below | Purpose |
| --- | --- | --- | --- | --- |
| L1 | F.Cu | 1 oz | **0.2104 mm** prepreg, εr ≈ 4.05 | All components, RF trace, short runs |
| L2 | GND | 0.5 oz | 1.065 mm core | **Solid unbroken ground plane** |
| L3 | PWR | 0.5 oz | 0.2104 mm prepreg | 3V3, 5V0, V_MEM polygons |
| L4 | B.Cu | 1 oz | — | Secondary routing + ground pour |

The **0.21 mm L1–L2 gap is the point**: it makes 50 Ω come out at a routable 0.33 mm, and keeps every L1 signal's return loop tiny.

### Impedance

| Net | Target | Geometry | Controlled? |
| --- | --- | --- | --- |
| RF to U.FL | 50 Ω single-ended | 0.33 mm microstrip L1 over L2 | No — keep it under 25 mm and the standard stackup is within a few % |
| USB D+/D− | 90 Ω differential | 0.3 mm traces, 0.2 mm gap | No |

Do **not** pay for controlled impedance. Set the stackup in KiCad (Board Setup → Physical Stackup), use the built-in calculator, keep the trace short.

Inner 0.5 oz copper is fine: peak current is under 700 mA.

---

## 3. Floorplan

```
┌──────────────────────────┬─────────────────────────┐
│ GNSS SAM-M10Q            │ Delock 12592 (868)      │
│ sky-facing, no copper    │ U.FL ──► board edge     │
│ under the patch          │ ◄─ RF keep-out ─►       │
├──────────────────────────┴─────────────────────────┤
│ IMU   │ MS5611  │                                  │
│ mech  │ vented  │       STM32F407VET6              │
│ centre          │       LQFP-100                   │
├──────────────────────────┬─────────────────────────┤
│ QUIET ZONE               │ microSD │ SWD │ USB      │
│ V_MEM LDO + INA226       │                         │
│ SRAM FRAM NOR MRAM       │ AS7331 ▲ sky aperture   │
├──────────────────────────┼─────────────────────────┤
│ NOISY ZONE               │ Connector edge          │
│ TPS62933 buck            │ battery · LR900-A GH1.25│
│ TPS63020 buck-boost      │ SPS30 · Geiger · PT1000 │
└──────────────────────────┴─────────────────────────┘
```

| Rule | Why |
| --- | --- |
| Switchers one corner, memory experiment the opposite corner | Distance is the cheapest isolation. M7 looks for millivolt anomalies. |
| GNSS patch: clear sky, no copper beneath | Per the u-blox integration manual |
| 868 antenna as far from GNSS as possible | 20 dBm at 868 next to a receiver at 1575 MHz is the classic self-jam |
| MS5611 near a shaded vent, not in SPS30 airflow | Its port needs ambient pressure; it is the independent variable for four missions |
| IMU at the mechanical centre | Off-centre turns rotation into apparent acceleration |
| MCU central | Everything routes to it |
| All external connectors on one edge | Cable management in a 3U tube |
| Crystals against the MCU, guard ring, nothing underneath | High-impedance node, keep the loop tiny |

---

## 4. Grounding — one plane, no splits

**L2 is ground and nothing else. No splits, no moats, no star grounds, no separate analogue ground.**

### Return current

Above ~100 kHz, impedance is dominated by **inductance**, and inductance scales with loop area. So return current crowds into the plane **directly under the trace**. Mental model: **every L1 trace has an invisible twin on L2.** Never make the twin detour.

### Why splitting is a trap

The "split analogue and digital ground, join at one point" advice comes from 1980s low-frequency instrumentation. On a board with a 168 MHz MCU, 24 MHz SPI and an 868 MHz radio it is harmful: a trace crossing a slot forces its return around the slot end, creating exactly the large loop the split was supposed to prevent.

**Partition by placement, not by copper cuts.** That is what the floorplan does.

### Rules

1. L2 is ground only. If you want to route "just one trace" there, use L4.
2. Never cross a plane gap. After routing, display **L2 alone** and look for holes.
3. Pour ground on L4, stitch to L2 every 5–10 mm.
4. Every IC ground pin gets its **own** via, as close to the pad as the rules allow.
5. Thermal pads get a 4–9 via array straight into L2.
6. The **only** deliberate copper gap is the Geiger optocoupler isolation barrier.

---

## 5. RF routing

**Your entire RF responsibility is one trace about 15 mm long.** Both modules have a pre-matched, certified front end.

### Why digital rules are insufficient

At 868 MHz in FR-4, λ ≈ **175 mm**. Transmission-line behaviour matters above ~λ/10, so **anything over ~17 mm is a transmission line**. Then: impedance is set by geometry not resistance; a mismatch reflects power instead of transmitting it; and bends, stubs, vias and plane gaps are all discontinuities.

### 50 Ω on this stackup

**0.33 mm (13 mil) wide microstrip on L1 over L2.** Verify in KiCad: Tools → Calculator Tools → Transmission Lines → Microstrip.

### Microstrip, not CPWG

Use plain microstrip. CPWG adds a third parameter (the gap) that must be held accurately and needs a dense via fence — more ways to get it wrong, no benefit at 868 MHz over 15 mm. If ground pour ends up beside the trace anyway, **keep it ≥ 3× trace width away (~1 mm)** or you have accidentally built a badly-dimensioned CPWG.

### The eight rules

1. Keep it **under 25 mm**.
2. **Constant width** the whole way — no necking between pads.
3. **No vias.** Entirely on L1, pad to pad.
4. **No right angles.** Two 45° bends or an arc.
5. **Solid L2 ground beneath the entire trace.** Verify by displaying L2 alone.
6. **Stitching vias both sides**, ~1 mm from the trace edge, 2–3 mm spacing (well under λ/20 ≈ 9 mm).
7. **3 mm keep-out** on all layers: no traces, no components, no L3 power under it — pour ground on L3 there instead.
8. **U.FL footprint from the manufacturer drawing**, ground pads vias straight down at the pad.

### Copy vs design

| Copy exactly from the manufacturer | Yours to design |
| --- | --- |
| Module land pattern and pad dimensions | The 50 Ω trace from RF pad to connector |
| **Ground and copper keep-out areas** around the module | Ground pour and stitching around that trace |
| Recommended ground via placement under the module | Where the module sits |
| Supply decoupling values and placement | Which regulator feeds it, ferrite + bulk |
| Any antenna-port matching network shown (usually none) | Connector choice and cable to the antenna |
| Recommended PCB edge clearance | Everything outside the keep-out |

**Design nothing inside the module's keep-out.** That geometry is what the certification was tested with.

---

## 6. Power routing

### The hot loop

Path: input cap → high-side switch → inductor → output → ground return → input cap. Current changes by amps in nanoseconds. Loop area determines radiation.

1. **Input capacitor as close to VIN and GND pins as physically possible.** Same side, no vias between cap and pin. This placement matters more than everything else in the power section.
2. Keep the hot loop on **one layer**.
3. Solid ground beneath the regulator, stitched at the IC ground pad.
4. **Switch node = a small fat polygon**, not a trace. Low resistance but **minimal area** — it is the noisiest node on the board.
5. Nothing sensitive under or beside the switch node.
6. **Feedback trace** is the quiet one: short, away from SW and the inductor, connected at the **output capacitor** not the inductor.

### Decoupling

Effectiveness is limited by the **loop inductance from cap to pin and back**, not by capacitance. So placement dominates:

- One 100 nF per supply pin, within 2 mm
- Cap ground pad gets its **own** via to L2
- Same side as the IC
- 4.7–10 µF bulk per cluster, placement less critical
- 0603 not 0402 — free with PCBA, reworkable

**STM32F407 has 11 VDD pins.** One 100 nF each, plus bulk, plus VDDA = ferrite + 1 µF + 100 nF.

### L3 polygons

**3V3** (largest), **5V0** (reaching the LR900-A, SPS30, Geiger connectors), **V_MEM** (small isolated island under the memory group only). ≥0.5 mm between polygons. Do **not** pour ground on L3 except where no rail needs the space — e.g. under the RF trace.

Power traces **1 mm minimum** (4× margin at 1 oz), which also keeps IR drop negligible — important because the pack already sags to 4.4 V.

---

## 7. Sensitive zones

**V_MEM island (M7)** — TLV75801 + memories + INA226 as one tight cluster, farthest corner from both switchers. Own L3 polygon, not touching 3V3. π-filter the LDO input (ferrite + 22 µF each side). DAC→FB trace short, direct, on L1 over solid ground, away from SPI. INA226 sense lines = a differential pair with a **Kelvin connection at the shunt's own pads**. 220 Ω isolation resistors **at the MCU end**.

**Analogue front end** — VDDA via ferrite + 1 µF + 100 nF, own quiet plane corner. VREF+ to VDDA through the same filter, never to digital 3V3. PT1000 reference resistor 0.1% 25 ppm, away from the regulators. PT1000 leads are long and outside the payload, so they are an antenna for the 868 transmitter — RC or ferrite filter plus ESD at board entry.

**Geiger HV interface** — 400 V stays on the external module. 6N137 optocoupler is the only path. **≥2 mm creepage** and a copper keep-out slot under the optocoupler body — the one place a deliberate plane gap is correct. Keyed 3-pin connector. Label the voltage on the silkscreen.

---

## 8. KiCad design rules (JLCPCB standard 4-layer)

| Constraint | Set to | JLC min |
| --- | --- | --- |
| Track width | 0.15 mm | 0.09 |
| Clearance | 0.15 mm | 0.09 |
| Via hole | 0.3 mm | 0.15 |
| Via annular ring | 0.15 mm (0.6 pad) | 0.13 |
| Hole to hole | 0.5 mm | 0.5 |
| Track to hole | 0.25 mm | 0.2 |
| Silkscreen width | 0.15 mm | 0.15 |
| Silkscreen text | 1.0 mm min | — |
| Edge to copper | 0.3 mm | 0.2 |

### Net classes

| Class | Width | Clearance | Via | Nets |
| --- | --- | --- | --- | --- |
| Default | 0.2 | 0.15 | 0.3/0.6 | unassigned |
| Power | **1.0** | 0.2 | 0.4/0.8 | 3V3, 5V0, VBAT |
| RF | **0.33** | **1.0** | none | the 868 net |
| USB | 0.3 / 0.2 gap | 0.2 | 0.3/0.6 | USB_DP, USB_DM |
| Analog | 0.25 | 0.3 | 0.3/0.6 | VDDA, VREF, DAC_OUT, PT1000, INA sense |
| V_MEM | 0.5 | 0.3 | 0.3/0.6 | the margined rail |

The **RF class 1.0 mm clearance** enforces the keep-out automatically — DRC will not let you forget.

### Custom DRC rules

```
(version 1)

(rule "RF no vias"
  (condition "A.NetClass == 'RF' && A.Type == 'Via'")
  (constraint disallow via))

(rule "RF keepout from other copper"
  (condition "A.NetClass == 'RF' && B.NetClass != 'RF'")
  (constraint clearance (min 1.0mm)))
```

### Manufacturing settings

- **ENIG, not HASL.** Flat surface matters for the LGA-14 IMU and QFN baro. A few dollars on 5 boards, removes a whole class of defect.
- Green mask is easiest to read while debugging.
- Vias tented except test-point vias.
- No castellated holes, no edge plating, no blind/buried vias.
- Impedance control **off**.

---

## 9. Workflow

**Phase 1 — before drawing (1–2 days).** Board outline and mounting holes on Edge.Cuts first. Set up stackup, design rules, net classes. Collect every datasheet in one folder. Draw the power tree on paper.

**Phase 2 — schematic (4–6 days).** Hierarchical sheets: Power, MCU, Sensors, Memory, Radio, Connectors. Copy each regulator's reference schematic exactly. Use STM32CubeMX for pin assignment **first** — it catches peripheral conflicts before they reach the schematic. Note I²C addresses as schematic text. Then: run ERC and fix **everything**; check every footprint against the manufacturer land pattern (do not trust KiCad defaults for the IMU, GNSS, radio or connectors — footprint errors are the commonest cause of a dead first board); print it and read it on paper two days later.

**Phase 3 — layout (5–8 days).** Spend a full day on placement alone. Route power, then RF, then clocks and analogue, then buses, then the rest. Place decoupling as you route each IC. Pour L2, verify unbroken. Pour and stitch L4. Run DRC continuously. 3D view for collisions. Silkscreen: every connector pinout, every test point net, polarity marks, version and date.

**Phase 4 — review before ordering (1 day, do not skip).** Display L2 alone. Display the RF net alone. **Print the layout 1:1 and place the actual components on it.** Check every polarised part against pin 1. ERC and DRC clean. BOM against schematic, part by part. Verify every machine-placed part is in the JLC library and in stock.

**Total: ~12–17 working days to order.** Then 5–8 days fab+assembly, 4–8 working days shipping, plus customs.

---

## 10. Fabrication outputs

| File | Format | How in KiCad |
| --- | --- | --- |
| Gerbers | RS-274X per layer | File → Fabrication Outputs → Gerbers |
| Drill | Excellon, PTH + NPTH separate, mm, absolute origin | Same dialog → Generate Drill Files |
| BOM | CSV: Comment, Designator, Footprint, LCSC Part # | JLCPCB Fabrication Toolkit plugin |
| CPL | CSV: Designator, Mid X, Mid Y, Layer, Rotation | File → Fabrication Outputs → Component Placement |
| Assembly drawing | PDF | Plot F.Fab + B.Fab + silkscreen |
| Schematic | PDF | File → Plot |

Install the **JLCPCB Fabrication Toolkit** plugin (Plugin and Content Manager). It produces all four with correct settings in one click.

Gerber layers: F.Cu, In1.Cu, In2.Cu, B.Cu, F.Paste, B.Paste, F.Silkscreen, B.Silkscreen, F.Mask, B.Mask, Edge.Cuts. Subtract soldermask from silkscreen **on**, plot footprint values **off**, references **on**, 4.6 mm coordinates.

### The two things that go wrong on first orders

1. **CPL rotation.** KiCad and JLCPCB rotation conventions often differ by 90° or 180°. A part at the wrong rotation is assembled backwards. **Check every polarised part in JLCPCB's online assembly preview before paying.** The plugin handles most known offsets — verify anyway.
2. **Stock changing between quote and order.** JLCPCB stock is live. If a part sells out mid-order they substitute or halt the job.

### Order settings

Layers 4 · your outline · **qty 5** · 1.6 mm · impedance control No · stackup **JLC04161H-7628** · **ENIG** · 1 oz outer, 0.5 oz inner · vias tented · **PCBA yes, 5 boards, top side only** · Economic if all parts qualify · **Confirm parts placement: Yes**

Assembling all five costs little more than two because setup dominates. Five means you can destroy one in thermal testing, keep a known-good reference, fly one, hold a flight-day spare, and still have one to cut up for diagnosis.
