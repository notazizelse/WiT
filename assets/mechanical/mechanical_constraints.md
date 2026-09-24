# Mechanical constraints — 3U, 100 × 100 × 300 mm

Status: envelope **CONFIRMED** by user. Board outline **recommended below, needs your sign-off** before layout starts.

---

## 1. Width budget — where the 100 mm goes

This is the calculation that sets your board size. Two options, and they differ by 10 mm of board width.

### Option A — insulate on the OUTSIDE (recommended)

| Item | mm consumed | Remaining width |
| --- | --- | --- |
| External envelope | — | 100 |
| Structural wall, 3 mm each side (3D print / plywood / thin ply) | 6 | 94 |
| Clearance for cable routing and connector bodies, 2 mm each side | 4 | 90 |
| **Board width available** | | **90 mm** |

XPS foam is wrapped around the outside of the box. The 100 × 100 mm figure then refers to the structure, and the finished payload is ~150 × 150 mm with 30 mm foam. Check whether your mechanical lead intends 100 mm to be the *structure* or the *finished exterior* — it changes everything below.

### Option B — insulate on the INSIDE

| Item | mm consumed | Remaining width |
| --- | --- | --- |
| External envelope | — | 100 |
| Structural wall, 3 mm each side | 6 | 94 |
| XPS foam, 10 mm each side | 20 | 74 |
| Cable clearance, 2 mm each side | 4 | 70 |
| **Board width available** | | **70 mm** |

70 × 70 mm is tight for this component count. It is doable but the floorplan gets cramped, the RF and memory zones lose their separation, and routing gets harder.

**Recommendation: Option A.** Insulating outside costs nothing and buys you 20 mm of board width, which is the difference between a comfortable layout and a difficult one.

---

## 2. Recommended board outline

```
Board:          90.0 × 90.0 mm, rectangular, 2 mm corner radius
Mounting holes: 4 × Ø3.2 mm (M3 clearance)
Hole positions: 80 × 80 mm square, i.e. 5 mm in from each edge
Keep-out:       6 mm radius around each hole (standoff footprint)
Thickness:      1.6 mm
```

**Alternative if you want CubeSat stackability:** the PC/104 form factor used across the CubeSat industry is 90.17 × 95.89 mm with holes on an 82.55 × 88.90 mm pattern. Adopting it lets you buy off-the-shelf CubeSat standoffs and stack future boards. Slightly more awkward numbers, real long-term benefit. Your call.

---

## 3. Height budget — the 300 mm

The tallest single item is the **SBM-20 Geiger tube at 108 mm long**, which does not fit horizontally in a 94 mm internal width. It must be mounted **vertically**.

| Zone (from top) | mm | Contents |
| --- | --- | --- |
| 0 – 45 | 45 | **Sky-facing deck.** GNSS patch with clear view up. AS7331 under the quartz/PTFE window. No metal or copper above either. |
| 45 – 90 | 45 | **Main PCB** with standoffs and tall components (radio modules, electrolytics, SD socket, connectors). |
| 90 – 145 | 55 | **Battery pack.** 4 × AA in a 2 × 2 holder ≈ 29 × 29 × 51 mm. Mount close to the payload's centre of mass. |
| 145 – 265 | 120 | **SBM-20 tube, vertical**, plus its HV board (~60 × 30 mm) beside it. 400 V — physically fenced off from everything else. |
| 265 – 300 | 35 | **SPS30** (41 × 41 × 12 mm) with its air inlet and outlet vented to outside air, plus the parachute attachment and cable exit. |

Below the payload, hanging free: the **868 MHz half-wave dipole** (16.4 cm total) and the **PT1000 probe on its 300 mm lead**.

### Placement rules that come from this layout

- **MS5611 vent path** must reach outside air, be shaded, and be away from the SPS30 fan outlet. A small labyrinth vent on a side face works.
- **868 MHz antenna and GNSS patch at opposite ends.** A 20 dBm transmitter at 868 MHz beside a receiver listening at 1575 MHz is the classic self-jamming failure.
- **Geiger tube far from the battery and the radios** — not for RF reasons but because it is the one high-voltage item and it should be reachable for inspection without disturbing anything else.
- **IMU as close to the payload's mechanical centre as possible.** Off-centre mounting turns rotation into apparent linear acceleration. Note the offset in firmware if you cannot centre it.

---

## 4. Mass budget

| Item | g |
| --- | --- |
| Assembled PCB | 80 |
| 4 × Energizer L91 AA | 58 |
| Battery holder | 25 |
| SPS30 | 40 |
| Geiger module + SBM-20 tube | 60 |
| MicoAir LR900-A + antenna | 25 |
| Cables, connectors, pigtails | 40 |
| Independent recovery tracker | 60 |
| Structure (100 × 100 × 300 box) | 250–400 |
| XPS insulation | 80 |
| Parachute, line, swivel | 100 |
| **Total** | **~820–970 g** |

Under 1 kg. Comfortably inside the common regulatory threshold for unregulated balloon payloads (often 4 kg), but **confirm the Uzbek requirement** with the same authority handling your frequency approval — a balloon payload is an aviation matter as well as a spectrum one.

---

## 5. Thermal design

With **~1 W of dissipation** inside a box insulated with 20–30 mm of XPS, the interior typically sits **30–50 °C above ambient**. At a −60 °C ambient the electronics likely see **−10 to −25 °C**, not −60 °C.

Design consequences:

- Design the **board** for −25 °C operation with margin, not for −60 °C.
- Design the **external PT1000 probe** and the **battery** for the real −60 °C.
- **Measure internal against external temperature** and log both. Validating this prediction is itself a small, clean engineering result worth a graph.
- Do **not** add a heater in v1. Add the two temperature sensors, fly, and decide from data. A heater adds current, complexity and a failure mode for a problem you have not yet confirmed exists.

---

## 6. Still needs your confirmation

1. Does **100 × 100 mm** mean the structural box, or the finished insulated exterior?
2. Is the structure a **tube or a square box**, and what material and wall thickness?
3. Is the payload **rigid-mounted** to the balloon train, or does it hang on a swivel? (Affects spin rate and therefore the M6 antenna-pointing analysis.)
4. **Where does the parachute attach**, and does its hard point pass through or beside the board stack?
