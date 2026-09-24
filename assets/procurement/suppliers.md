# Procurement — where to order, from Uzbekistan

Three tiers. Use local first where quality is not critical, authorised distributors for anything that must be genuine, and JLCPCB/LCSC for the board and its machine-placed parts.

---

## Tier 1 — Local, Tashkent

Fastest, no customs, no shipping. Good for passives, connectors, tools, generic modules and anything you will buy several of.

| Supplier | URL | What they carry | Use for |
| --- | --- | --- | --- |
| **Microelectronica (electron.uz)** | https://electron.uz/ | Broad electronic components, sensors, modules, tools, Raspberry Pi. Published price list. | Passives, connectors, headers, tools, multimeter, tweezers, flux, ESD gear, generic modules |
| **arduino.uz** | https://arduino.uz/ | Arduino/Raspberry ecosystem, sensor modules, breakouts | Sensor breakout boards for dev-board prototyping |
| **NAG Uzbekistan** | https://shop.nag.uz/ | Sensors and industrial components, Tashkent stock | Industrial sensors, cabling |
| **Nevel.uz** | https://nevel.uz/elektronnye-komponenty-174 | General electronic components | Passives, semiconductors |
| **OLX.uz** | https://www.olx.uz/ | Second-hand and small-trader listings | Bench PSU, oscilloscope, freezer — used lab gear |
| Retail (supermarkets, photo shops) | — | **Energizer Ultimate Lithium AA (L91)** | **CONFIRMED available locally.** Buy 8 cells (two flight sets). |

**What NOT to buy locally:** the STM32, the IMU, the GNSS module, the MS5611, or any memory IC. Counterfeit and remarked parts circulate through informal channels, and a counterfeit IMU that half-works is the worst possible debugging experience. Those go through Tier 2 or Tier 3.

---

## Tier 2 — Authorised distributors (genuine parts, ships worldwide)

Use for anything where authenticity matters or where the part is not in the JLCPCB library.

| Supplier | URL | Ships to UZ | Notes |
| --- | --- | --- | --- |
| **Mouser** | https://www.mouser.com/ | Yes, DHL/FedEx | Best for Sensirion (SHT45, SPS30), ams OSRAM (AS7331), Fujitsu FRAM, Everspin MRAM, u-blox, Samtec. Free shipping threshold applies in many regions — check at checkout. |
| **DigiKey** | https://www.digikey.com/ | Yes — [international shipping rates](https://www.digikey.com/en/help-support/delivery-information/international-shipping-rates) | Same catalogue coverage as Mouser. Compare both; stock and price differ per line. |
| **LCSC** | https://www.lcsc.com/ | Yes — [shipping cost calculator](https://www.lcsc.com/faqs/shipping-delivering/shipping-cost-calculation) | Cheapest for the STM32, ICM-42688-P, MS5611, W25Q128, passives. Same warehouse as JLCPCB, so ordering loose spares alongside a PCBA order is efficient. |
| **Reichelt / Digi-Key EU** | https://www.reichelt.de/ | Yes | Carries Delock. Delock is a German brand — the EU distributors are the reliable route. |

**Customs.** Expect Uzbek import duty and VAT on the declared value. Budget ~15–25 % on top of goods value plus shipping, and allow extra calendar days for clearance. Consolidate orders: three shipments of $300 cost far more in fees and delay than one of $900. **This is a real schedule item, not an afterthought** — put it in the plan.

**Practical tip:** if your university or a sponsor can act as the importer, clearance is usually faster and duty treatment may differ for educational equipment. Ask before you order.

---

## Tier 3 — AliExpress

Confirmed acceptable to you for the Geiger module. Reasonable for a specific, limited category.

**Fine to buy here:**
- RadiationD-v1.1 / CAJOE Geiger module with SBM-20 tube (**confirmed orderable**)
- U.FL and SMA pigtails, SMA bulkheads, antennas
- STM32F407VET6 "black board" dev boards
- Logic analyser, tweezers, ESD mat, standoffs, Kapton tape
- PT1000 probes on leads
- 868 MHz Yagi

**Do not buy here:**
- Any IC that goes on the PCB
- The u-blox GNSS module (heavily counterfeited)
- The IMU (heavily counterfeited)
- microSD cards (fake capacity is endemic)

**Lead time:** 2–5 weeks to Tashkent. Order AliExpress items **first**, because they are the slowest and the least predictable.

---

## Tier 4 — JLCPCB (board + assembly)

| Item | Detail |
| --- | --- |
| URL | https://jlcpcb.com/ |
| Order | 5 boards, 4-layer, 1.6 mm, stackup **JLC04161H-7628**, **ENIG** finish, PCBA top side only |
| Parts | Everything in `BOM_pcb_jlcpcb.csv` — all have LCSC numbers |
| Excluded | Everything in `BOM_handsolder.csv` — you fit those yourself |
| Files | Gerbers, Excellon drill, BOM CSV, CPL CSV — use the **KiCad JLCPCB Fabrication Toolkit** plugin |
| Lead time | 5–8 days fab + assembly, then 4–8 working days DHL to Tashkent, plus customs |
| **Before paying** | Use JLCPCB's **online assembly preview** and check pin-1 orientation on every polarised part. KiCad and JLCPCB rotation conventions disagree; this is the single most common first-order failure. |

**Verify stock before freezing the layout.** JLCPCB stock is live. If a part sells out mid-order they substitute or halt the whole job.

---

## Order sequence — what to buy today

Nothing in group 1 depends on the PCB being finished, and every item in it removes a schedule risk.

**Today**

| # | Item | Supplier | Why now |
| --- | --- | --- | --- |
| 1 | STM32F407VET6 dev board × 2 | AliExpress | Firmware development starts immediately and runs parallel to everything else. Highest-value $24 in the project. |
| 2 | Geiger module + SBM-20 × 1 | AliExpress | Longest and least predictable lead time. |
| 3 | u-blox SAM-M10Q × 2 | Mouser / DigiKey | Longest distributor lead-time risk, and you need one on the bench to write and test the UBX airborne-mode config. |
| 4 | MicoAir LR900-A × 2 | MicoAir / distributor | Stock already known to be uncertain. Start the clock. |
| 5 | Bench PSU + ST-LINK × 2 + multimeter + logic analyser | electron.uz / OLX.uz / AliExpress | Needed **before** the board arrives, not after. |

**This week**

| # | Item | Supplier |
| --- | --- | --- |
| 6 | Sensor breakouts: MS5611, ICM-42688-P, SHT45, AS7331, TMP117 | AliExpress + Mouser |
| 7 | Bare parts ×4 each: MS5611, ICM-42688-P, SHT45, AS7331, TMP117, FRAM | Mouser / LCSC |
| 8 | Delock 12592 × 2 (flight + spare), 12593 × 1 (ground) | Reichelt / Digi-Key |
| 9 | SPS30 × 1 | Mouser / DigiKey |
| 10 | Energizer L91 AA × 8, holders × 2 | Local Tashkent retail |
| 11 | Quartz or PTFE UV window | Thorlabs / AliExpress / local optics |

**Once the layout is frozen**

| # | Item | Supplier |
| --- | --- | --- |
| 12 | JLCPCB: 5 boards assembled + stencil | JLCPCB |
| 13 | Loose spares ×2 of every hand-soldered part | LCSC (same shipment as 12 if possible) |
| 14 | microSD industrial × 2 | Mouser / DigiKey |

---

## Why order sensors before the PCB exists

Every sensor has a driver to write, a calibration to verify and a quirk to discover. Doing that on a dev board with breakouts, while the PCB is being laid out and fabricated, turns the fabrication wait from dead time into the bulk of your firmware schedule.

By the time the boards land you should be flashing known-good firmware and hunting hardware faults — not writing an I²C driver for the first time with a deadline behind you.
