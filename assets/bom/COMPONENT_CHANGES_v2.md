# Component list v2 — solidified for availability

Goal: remove every part whose supply chain is a single thin thread. Eight changes.
Nothing that carries a mission was weakened; two things got structurally simpler.

---

## Summary of changes

| # | Item | v1 | **v2** | Reason |
| --- | --- | --- | --- | --- |
| 1 | **Power architecture** | Buck 3V3 **+** buck-boost 5V | **Buck-boost 5V only, then LDO → 3V3** | One switcher instead of two. Halves the hardest layout task on your first 4-layer board. |
| 2 | 3V3 regulator | TPS62933 buck | **TLV75733PDQNR LDO** (WSON-6, thermal pad) | No inductor, no hot loop, quiet rail for M7 |
| 3 | On-board radio | Delock 12592 (SX1276, 862–893) | **Ebyte E22-900T22S** (SX1262, **850–930**) | One part covers 868 **and** 915. Newer chipset, far wider stock, ~1/3 the price. |
| 4 | Magnetometer | MMC5983MA (MEMSIC) | **LIS3MDL** (ST) | ST part, enormously more available, drivers everywhere |
| 5 | FRAM | MB85RS2MT (Fujitsu) | **CY15B102QN / FM25V20A** (Infineon) | Better Western distributor stock; Fujitsu stays as the alternate |
| 6 | MRAM | MR25H40, populated | **Footprint only, DNP** | Erratic stock, $26, least informative of the four technologies |
| 7 | SWD header | Samtec FTSH-105 ($2.60) | **Generic 2×5 1.27 mm SMD** ($0.30) | Identical function, no lead time, 1/9 the price |
| 8 | Barometer | 1 × MS5611 | **MS5611 + MS5607** (2nd CS on the same SPI bus) | The single point of failure for four missions, for $10 |

**Net effect on cost:** roughly **−$18 per board**. Availability risk drops substantially.

---

## 1–2. Power: one switcher, not two

### The change

```
v1:  Battery ──┬──► BUCK (TPS62933) ──► 3V3
               └──► BUCK-BOOST (TPS63020) ──► 5V0

v2:  Battery ──► BUCK-BOOST (TPS63020) ──► 5V0 ──┬──► LR900-A, SPS30, Geiger
                                                  └──► LC filter ──► LDO ──► 3V3
```

### Why

A switching regulator's **hot loop** — input cap, high-side switch, inductor, ground return — is the highest-di/dt structure on the board and the layout task most likely to go wrong for someone doing this for the first time. **v1 asked you to get it right twice.** v2 asks once.

Second benefit: the 3.3 V rail now comes from a linear regulator, which is inherently quiet. That rail feeds the MCU, the sensors and the memory experiment. M7's entire credibility rests on being able to say the rail was clean.

### The honest trade-off

| | v1 (two switchers) | v2 (switcher + LDO) |
| --- | --- | --- |
| Overall efficiency | ~88 % | ~62 % |
| Pack draw, average | ~125 mA | ~188 mA |
| 8-hour mission consumption | ~1000 mAh | ~1500 mAh |
| Margin against 3000 mAh pack | 3.0× | **2.0×** |
| Hot loops to lay out | 2 | **1** |
| Inductors | 2 | 1 |
| 3V3 rail noise | switcher ripple | **LDO-quiet above the filter corner** |

**2× margin is comfortable.** You have a large battery precisely so you can spend some of it on simplicity.

### One caveat you must handle

LDO power-supply rejection falls off above roughly 100 kHz–1 MHz, and the TPS63020 switches at ~2.4 MHz. The LDO alone will **not** remove switching ripple at that frequency.

**Fix:** an LC filter between the 5 V rail and the LDO input — a 2.2 µH shielded inductor (or a 600 Ω ferrite) plus 22 µF on each side. Cheap, small, and it puts the filter corner well below the switching frequency.

### Thermal check

LDO dissipation = (5.0 − 3.3) × I.

| Condition | Current | Dissipation | Rise (WSON-6 + copper pour, θJA ≈ 50 °C/W) |
| --- | --- | --- | --- |
| Average | 125 mA | 213 mW | ~11 °C |
| Peak (SD write + radio TX) | 350 mA | 595 mW | ~30 °C, transient |

Fine — and at stratospheric temperatures that 213 mW of waste heat is working *for* you, helping hold the interior above ambient.

**Package matters.** Use a **WSON-6 or SOT-223 with a thermal pad and a copper pour**, not a bare SOT-23-5. A SOT-23-5 at 595 mW would rise ~120 °C.

### Parts

| Ref | Part | Package | ~$ | Note |
| --- | --- | --- | --- | --- |
| U3 | TPS63020DSJR buck-boost → 5.0 V | SON-10 | 3.40 | unchanged from v1 |
| U2 | **TLV75733PDQNR** LDO → 3.3 V, 1 A | **WSON-6 with pad** | 0.85 | Alternates: TLV75733PDRVR, AP7365-33WG-7, MIC5504-3.3 |
| L1 | 2.2 µH 3 A shielded (buck-boost) | 4020 | 0.60 | value **from the TPS63020 datasheet**, not guessed |
| L3 | 2.2 µH 1 A shielded (LC filter) | 3015 | 0.30 | new |
| — | 22 µF X7R ×2 (LC filter) | 1210 | 0.30 | new |

Removed: TPS62933, its inductor and its output caps.

---

## 3. Radio: Ebyte E22-900T22S instead of Delock 12592

You invited a better alternative, and there is one.

| | Delock 12592 | **Ebyte E22-900T22S** |
| --- | --- | --- |
| Chipset | SX1276 | **SX1262** (newer, lower RX current, better sensitivity) |
| Frequency | 862–893 MHz | **850.125–930.125 MHz** |
| TX power | 10–20 dBm | 22 dBm |
| Interface | UART TTL 3.3 V | UART TTL, 3.3 V and 5 V compatible |
| Size | 26 × 16 × 3.2 mm | 16 × 26 mm, 2.1 g |
| Antenna | I-PEX MHF I | U.FL / IPEX |
| Availability | EU distributors (Delock is an accessory brand) | **Ebyte direct, Alibaba, Amazon, many resellers worldwide** |
| Price | ~$32 | **~$12** |

**The decisive argument is the band range.** 850–930 MHz covers **both** the 868 outcome and the 915 outcome of your frequency council decision, in one part number, with no board change. Given that the ruling is still pending, that is real insurance rather than a nice-to-have.

Second argument: Delock is a German consumer-accessory brand that resells a module. Ebyte is the module manufacturer, sells direct, and is stocked by a dozen channels including ones that ship to Uzbekistan quickly. For "no weak bones," that is the stronger supply chain.

**If you prefer to keep the Delock**, it remains perfectly good silicon — but the footprints are not compatible, so decide before layout starts. The E22 is what I have put in the v2 BOM.

The **MicoAir LR900-A is unchanged**, external on its GH1.25 header and its own UART. Nothing about this change touches it.

⚠ The interlock rule is unchanged and still absolute: **never key both radios simultaneously.**

---

## 4. Magnetometer: LIS3MDL

MMC5983MA has better noise performance. LIS3MDL is an ST part stocked by every distributor on earth, with drivers in every framework.

The magnetometer's job here is spin rate and coarse heading for the M6 antenna-orientation analysis. **LIS3MDL is far more accurate than that job requires.** Trading unneeded performance for supply certainty is exactly the right call on an optional part.

Alternates if even that slips: QMC5883L (very cheap, very common) or MMC5983MA (the v1 choice).

---

## 5. FRAM: Infineon instead of Fujitsu

`CY15B102QN` / `FM25V20A` — 2 Mbit SPI FRAM, Infineon (formerly Cypress, formerly Ramtron). Same capacity, same interface, same ferroelectric physics, same role as the M7 non-volatile control.

Infineon FRAM has consistently better stock at Mouser and DigiKey than Fujitsu's MB85RS line. **Fujitsu MB85RS2MTAPNF stays in the BOM as the named alternate** — if it is in stock when you order, either is fine.

---

## 6. MRAM: footprint only, do not populate

MR25H40 was always optional. Its stock is erratic, it costs $26, and of the four memory technologies it is the least informative — it is the control that is *expected* to show nothing, and your SEU statistics already predict zero events in every device.

**Keep the SOIC-8 footprint on the board, mark it DNP.** If one turns up cheaply later, solder it in. Costs nothing to leave the pads.

---

## 7. SWD header: generic

Samtec FTSH-105 is $2.60 with distributor lead time. A generic 2×5 1.27 mm SMD header from LCSC is $0.30, in the JLCPCB library, and mechanically identical for a debug connector you will plug into perhaps fifty times.

Samtec is worth paying for on connectors that see thousands of cycles or need retention under vibration. A bench debug header is neither — and in flight nothing is plugged into it.

**Keep the 2.54 mm test points for SWDIO / SWCLK / NRST / GND as a fallback regardless.**

---

## 8. New: a second barometer

The MS5611 is the independent variable for **M1, M2, M3 and M4**. One sensor, four missions. If it fails, or arrives from a bad batch, or its port gets blocked by conformal coating, most of the science goes with it.

**Add an MS5607-02BA03 on the same SPI bus with its own chip select.** $9.

Why a *different* part number rather than a second MS5611: two identical sensors from the same reel share failure modes and calibration errors. MS5607 has the same 10–1200 mbar range and the same silicon family but different output coding — so agreement between the two is meaningful evidence, and disagreement tells you something real.

You also get a free cross-check dataset: two independent pressure measurements through the whole flight is its own small validation result.

**This is the only part v2 *adds*.** Everything else is a swap or a removal.

---

## What did NOT change, and why

| Part | Kept because |
| --- | --- |
| **STM32F407VET6** | Already the most available STM32 in its class. Massive community, cheap dev boards, in the JLCPCB library. No weakness to fix. |
| **MS5611** | No alternative exists that reaches stratospheric pressure. The mitigation is redundancy (#8), not substitution. |
| **SAM-M10Q** | Genuine u-blox, stocked at Mouser, DigiKey and LCSC. **Fallback: SAM-M8Q** — older, cheaper, even more widely stocked, same 15.5 × 15.5 LCC family, also supports the airborne dynamic model (via `UBX-CFG-NAV5` rather than `CFG-VALSET`). Verify pin compatibility before assuming a drop-in swap. |
| **ICM-42688-P** | In the JLCPCB library, widely stocked. Fallback: BMI088 or ASM330LHHX. |
| **SHT45** | Fine availability. Fallback: **SHT31-DIS** — more common, ±2 %RH instead of ±1 %, acceptable given humidity is a secondary channel. |
| **TMP117** | Mainstream TI. Fallback: MCP9808 (±0.25 °C — still adequate for the M7 retention curve). |
| **MAX31865** | Standard RTD front end, widely available. |
| **INA226** | Extremely common TI part. |
| **W25Q128JVSIQ** | One of the most-produced flash chips in existence. |
| **23LC1024** | Microchip, stable, long-term available. |
| **6N137** | Commodity optocoupler, available everywhere. |
| **AS7331** | **The one remaining thin link.** No true UVA/UVB/UVC alternative exists at this price — single-channel parts cannot produce the M4 result. Mitigation: buy two, and keep the SparkFun Qwiic breakout as a hand-wired fallback if the bare IC slips. |
| **SPS30** | Sole credible PM sensor at this quality. It is also the designated first cut if the schedule compresses, so the exposure is bounded. |

---

## Revised cost

| | v1 | v2 | Δ |
| --- | --- | --- | --- |
| PCB components per board | ~$186 | **~$168** | −$18 |
| Assembled board each (qty 5) | ~$229 | **~$211** | −$18 |
| 5 assembled boards | $1145 | **$1055** | −$90 |
| Project total | ~$2400 | **~$2300** | −$100 |

Cheaper, simpler to lay out, and with fewer single-source dependencies. The removals (second switcher, MRAM, Samtec) paid for the addition (second barometer) with change left over.
