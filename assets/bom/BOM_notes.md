# BOM notes — why each choice, and why not the alternatives

Companion to the CSVs. Every decision that is not obvious from a datasheet is recorded here.

## The five choices that matter most

### 1. MS5611, not BMP390 / BMP280 / BME280 — CRITICAL

**The BMP390's specified range is 300–1250 hPa. 300 hPa is about 9 km.** BMP388, BMP280 and BME280 are the same. Every one of them is the default recommendation in hobby altimeter tutorials, and every one is wrong for a stratospheric balloon: above 9 km they are out of specification and their readings are meaningless.

MS5611 is specified **10–1200 mbar ≈ 31 km**, 0.012 mbar resolution. There is no cheap alternative reaching stratospheric pressure. MS5607 is the same silicon with different output coding — buy two as a fallback.

Above ~31 km, pressure altitude is gone. GNSS altitude is the only reference. Log both and show the crossover in your analysis rather than switching silently.

**Layout:** open port. Must not be conformal-coated, taped, foamed or flux-contaminated. Shaded, and not in the SPS30 fan's airflow.

### 2. STM32F407VET6, not F411 / H743 / ESP32 — CRITICAL

Four reasons, only one of them performance:

1. **It has a DAC.** The M7 retention experiment needs a programmable rail. **The STM32F411 has no DAC at all** — the otherwise-obvious cheap choice is disqualified on this alone.
2. **4 KB battery-backed backup SRAM** in the VBAT domain, surviving brownouts. Error counters, reset log and mission state live there. Free.
3. **Hardware CRC32** — checksums megabytes per second instead of burning CPU.
4. **A €12 dev board exists.** 90% of the firmware gets written while the PCB is in fab. The largest schedule lever in the project.

Not H743: extra rails, SMPS configuration at bring-up, 3× the price, compute you have no use for without a camera. Not ESP32: Wi-Fi useless at altitude, poor ADC, weaker determinism. Not RP2040: no internal flash, no RTC domain, inadequate ADC for M7.

LQFP-100 at 0.5 mm is both machine-placeable and hot-air reworkable. QFN or BGA is neither.

### 3. SAM-M10Q, not MAX-M10S or a bare chip — CRITICAL

**Integrated patch antenna** removes an entire GNSS RF chain: no SAW, no LNA, no 50 Ω feed, no patch placement study, no ground-plane sizing. For a first RF-bearing PCB, deleting a whole RF subsystem for ~$10 is the best value in this BOM. LCC castellated, so hot-air friendly.

**Two gotchas that silently kill the mission:**

- **COCOM limit.** Most receivers stop producing a fix above 18 km unless set to the **airborne <1 g dynamic model** (`CFG-NAVSPG-DYNMODEL = 8`). That must be sent over UBX **at every boot from firmware** — not relied on from saved flash, which a brownout can lose. Eighteen kilometres is exactly where the science starts.
- **Route 1PPS to a timer input capture.** Absolute disciplined time is what lets you correlate a memory event, a radiation count and a ground packet to the same instant. One pin, free.

Avoid ultra-cheap ATGM336H-class modules: inconsistent airborne-mode support, and you cannot find that out at 18 km.

### 4. Energizer L91 AA ×4, not Li-ion — CRITICAL

**Confirmed from the product packaging: −40 °C to +60 °C.** It is the only common primary chemistry specified for this. 3000 mAh, 1.5 V, 14.5 g each. 4S gives 6.0 V nominal, 7.2 V fresh, ~4.4 V end-of-life. **Confirmed purchasable locally in Tashkent.**

Standard Li-ion is rated 0…+45 °C for discharge and collapses in the cold — it would need a heater, which is current, mass and a new failure mode. 1S LiPo + boost is worse still: the boost converter's noise sits directly on top of the M7 measurement.

Non-rechargeable, so **do all bench work on the lab supply.** J1 accepts either.

### 5. Two pre-certified radio modules, not a chip-down SX1262 — CRITICAL

A chip-down radio needs a matching network, a harmonic filter, crystal pulling analysis and a spectrum analyser to verify. Both modules here contain a matched, certified front end. **Your entire RF responsibility becomes one 50 Ω trace about 15 mm long.**

| | MicoAir LR900-A | Delock 12592 |
| --- | --- | --- |
| Role | Primary downlink | Independent backup + beacon |
| Chipset | LoRa | SX1276 |
| Band | 890–915 MHz | 862–893 MHz |
| TX power | 22 dBm | 10–20 dBm |
| Sensitivity | not published | −146 dBm @ 2.4 kbps |
| Supply | **4.5–5 V** (needs the 5V rail) | 3.3–5.2 V |
| Mounting | External, GH1.25-4P | **SMD onto your PCB** |
| Temp | not published | **−40…+85 °C** |
| TX current | "300 mW max" — **MEASURE IT** | 120 mA @ 20 dBm |

**Link budget:** Delock 20 dBm + −146 dBm = 166 dB. FSPL at 868 MHz over 300 km ≈ 141 dB. ~20 dB margin at 300 km slant range. The 30 dBm variant belongs on the **ground**, where power and cooling are free — not in the air where every 100 mA is battery mass.

**⚠ Never key both radios at once.** 868 and 890–915 MHz are close enough that a 20 dBm transmitter will desense or damage the other's front end from centimetres away. Firmware needs a hard interlock — a single "radio busy" flag both drivers must acquire — not a convention. Antennas in opposite corners, oriented perpendicular.

**Antenna:** half-wave dipole below the payload (2 × 8.2 cm at 868). No ground plane or counterpoise needed, predictable pattern, nulls point up at the balloon and straight down — neither at your station. A quarter-wave whip needs a counterpoise you will model incorrectly on a first attempt.

---

## Sensors

**ICM-42688-P** — gyro noise ~0.0028 °/s/√Hz, best in class; ±32 g and 32 kHz ODR give burst detection and parachute-shock capture free from a part you needed anyway. Alternatives: BMI088 (separate dies, drone-proven), ASM330LHHX (automotive, **−40…+105 °C** — pick it if the thermal analysis worries you). **Avoid MPU-6050**: obsolete and heavily counterfeited.

**SHT45** — justified for exactly two things, and claim no more: humidity through the troposphere below ~10 km, and **condensation/icing detection on descent**, a genuine failure mode that has destroyed other people's flight data. Polymer RH sensors are unusable in the stratosphere: at −60 °C absolute humidity is ~0 and response time stretches to hours.

**Three separate temperature measurements, three jobs.** SHT45 = internal air. **PT1000 on a lead outside the insulation** = true ambient, the M1 measurement, works to −200 °C with no semiconductor to freeze out — what real radiosondes use. **TMP117 + 2 NTCs** = board temperature, the M7 axis, where ±0.1 °C matters because the retention curve is plotted against it. Read the PT1000 through a **MAX31865**, not a divider into the STM32 ADC — a divider will not deliver the accuracy M1 needs for comparison against ERA5.

**AS7331** — three **calibrated** channels (UVA/UVB/UVC). Cheap single-channel parts (GUVA-S12SD, ML8511, discontinued VEML6075) give one poorly-defined broadband output you cannot turn into a defensible irradiance figure. **Requires a quartz or PTFE window** — acrylic, polycarbonate and window glass all block UVB and UVC almost completely. Behind normal plastic you measure nothing and do not know why.

**SPS30** — best-quality PM sensor at this price, but **Sensirion publishes no low-pressure specification**. The fan moves a volumetric flow and the mass-concentration algorithm assumes near-sea-level density; as pressure falls the calibration stops being valid, with no correction factor you can apply with confidence. **Valid below ~5 km. Powered down above 6 km.** That is where the public-health signal is anyway. Alternatives: SEN55 adds VOC/NOx indices (~$60) but those are relative not absolute, so weaker scientifically; PMS5003 is a third of the price and noticeably less consistent.

**Geiger: buy it, do not build it.** Do not design a 400 V flyback on your first PCB. The CAJOE module with an SBM-20 tube is complete: HV supply, anode resistor, pulse shaper. Your board contributes one **optocoupled** pulse input, a 5 V feed and one timer in external-counter mode. SBM-20 is Soviet-surplus and common across the CIS. ⚠ 400 V DC — physically separate, ≥2 mm creepage, no exposed pads. Tube is **108 mm long → mount vertically** in the 3U.

**Magnetometer is optional but cheap.** Spin rate and heading turn M6 into a much better result: plot RSSI against antenna orientation and show the dipole nulls appearing as the payload rotates.

---

## Memory (M7)

Four technologies because each fails differently — that is the point.

| Device | Physics | Role |
| --- | --- | --- |
| 23LC1024 SRAM | active feedback, volatile | **Primary subject.** Genuine measurable retention threshold that moves with temperature. |
| MB85RS2MT FRAM | ferroelectric polarisation | Control. No charge pump, bus-speed writes, 10¹³ endurance. |
| W25Q128 NOR | trapped charge | Subject **and** backup flight log if the SD fails — hence CRITICAL not HIGH. |
| MR25H40 MRAM | magnetic | The control that *should* show nothing. Optional at $26. |

Plus free from the MCU: 192 KB internal SRAM, 512 KB Flash (CRC32-checked — the internal-vs-external comparison), and 4 KB backup SRAM for counters and reset logs.

**The SD card is not a test subject.** It contains a controller with internal ECC and wear levelling, so anything you measure is the controller's behaviour. Excellent logger, useless experiment. And SD cards are a leading cause of lost balloon data: they corrupt the filesystem on power loss. **Write a raw circular log to fixed sectors, not FAT**, and mirror critical telemetry to the W25Q128. Industrial −40 °C cards only — consumer cards are not specified below 0 °C.

**Why 220 Ω series resistors on SPI.** During the hold phase V_MEM may be at 1.2 V while the MCU is at 3.3 V. Driving 3.3 V into a chip powered at 1.2 V forward-biases its ESD structures and can latch it up. Hi-Z plus series limiting is the cheap correct fix. Place them **at the MCU end** so the whole trace is protected. This is also why the memory is on **SPI not a parallel FSMC bus** — four signals to isolate instead of twenty-eight.

**Set expectations on bit flips.** ~200 FIT/Mbit × 8 Mbit × 500× flux × 2 h above 15 km ≈ **1.6 × 10⁻³ expected upsets** — about one chance in six hundred. Expect zero and report an **upper bound**. Claiming a single anomaly was radiation, when a rail dip, EMI, temperature or a firmware bug are each far more likely, is what gets a student result dismissed. The INA226s, the reset forensics and the seed-regenerated patterns exist precisely so the null result is a *measurement*.

---

## Power

**Two switchers, not one.** The LR900-A needs 4.5–5 V. A plain buck cannot hold 5 V once the pack sags below ~5.5 V, which it does well before the cells are exhausted. A buck-boost on the 5 V rail lets you use the whole pack.

**The adjustable LDO (TLV75801) generates V_MEM** — the programmable rail the STM32 DAC sweeps to find each memory's retention threshold. This one part is the M7 experiment.

**Two INA226s, not one.** 3V3 main and V_MEM. This is not housekeeping. When M7 records a bit error the first question is "did the rail dip?" Without a 16-bit rail measurement logged alongside, the answer is a guess and the result is worthless.

**Real fuse, not PPTC.** PPTC hold and trip currents shift badly at −40 °C and may not protect when you need it.

**Polymer bulk caps, not aluminium electrolytic.** Electrolytic ESR rises sharply below −20 °C.

**X7R everywhere, C0G for crystal loads.** Y5V loses most of its capacitance at temperature extremes. A crystal load capacitor that drifts pulls your clock in the cold, which breaks UART framing and timestamp accuracy — hence C0G there specifically. Every cap is rated at least 2× its working voltage because ceramics lose 50–80 % of their capacitance under DC bias near their rating.

**Thermal reality.** ~1 W inside a box with 20–30 mm XPS typically holds the interior **30–50 °C above ambient**. The electronics probably see −10 to −25 °C, not −60 °C. Design the board for −25 °C with margin; design the external probe and the battery for the real −60 °C; and **measure internal vs external** to prove it. No heater in v1 — it adds current, complexity and a failure mode for a problem you have not confirmed exists.

---

## Assembly split

**JLCPCB assembles everything SMD; you hand-fit a deliberate list.** The critical path is one international shipment round-trip, so the goal is maximising the chance the first board works, not minimising any single step.

Hand-soldering the ICM-42688-P (LGA-14, 2.5 × 3 mm) without stencil and hot air is a real risk, and a marginal IMU joint is invisible and costs days. The MS5611 is QFN-8 with a vent hole that must not fill with paste. Machine paste printing handles both; a hand blob does not.

**Hand-solder list** (so you can swap them): both radio modules, the GNSS module, SWD header, all connectors, option jumpers, and anything not in the JLC library. You have a hot-air station, which is exactly what LCC and castellated modules need.

**Order 5 boards**, plus 2 loose sets of every hand-soldered part. Five means you can destroy one in thermal testing, keep a known-good reference, fly one, hold a flight-day spare, and still have one to cut up.

**0603 not 0402** — free with PCBA, and rework stays possible.

---

## Test points are not optional

Twenty-four of them, under one dollar total: every rail, NRST, BOOT0, 1PPS, every chip select, V_MEM, the Geiger pulse, both radio TX/RX pairs. A board without them turns a two-hour debug session into a two-day one.
