# WiT: 3U stratospheric balloon payload

The flight computer for a 3U (100 × 100 × 300 mm) payload that goes up on a stratospheric balloon. It's one 4-layer, ~90 × 90 mm board around an STM32F407, with two 900 MHz radios for telemetry and a sensor set chosen so that every sensor answers a specific question about the flight.

**Status: in design.** The architecture, parts list, power budget, stackup and test plan are done (in [`assets/`](assets/)). Schematic capture in KiCad is next, so the files in [`wit/`](wit/) are still empty.

## Missions

Each one is written up in [`missions/mission_definitions.md`](assets/missions/mission_definitions.md) as a hypothesis, the graph that would test it, and what would prove it wrong.

| | Mission | Main sensors |
|---|---|---|
| M1 | Upper-air temperature, pressure, humidity and wind profile over Uzbekistan, compared against ERA5 reanalysis | PT1000 + MAX31865, SHT45, MS5611, SAM-M10Q GNSS |
| M2 | How high Tashkent's polluted layer goes, and whether a temperature inversion caps it | SPS30, PT1000 |
| M3 | Cosmic radiation dose from the ground to burst | SBM-20 Geiger tube |
| M4 | UV irradiance against altitude | AS7331 |
| M5 | GNSS availability and accuracy against altitude | SAM-M10Q |
| M6 | LoRa link budget against range and altitude | Both radios |
| M7 | How off-the-shelf memory chips (SRAM, FRAM, NOR, MRAM) hold up under cold and low supply voltage | Memory under test + INA226s |

## Design decisions

| | |
|---|---|
| MCU | STM32F407VET6 (LQFP-100), run at 84 MHz |
| Radios | MicoAir LR900-A and Ebyte E22-900T22S on separate UARTs |
| Board | 4 layers (JLC04161H-7628), solid ground on L2 so the RF trace can be a 0.33 mm 50 Ω microstrip |
| Power | 4 × Energizer L91 AA (rated to −40 °C) → buck-boost to 5 V → LDO to 3.3 V. ~125 mA average, ~3× energy margin for an 8-hour flight |
| Firmware | Bare metal with a cooperative scheduler instead of FreeRTOS: no task blocks, all bus traffic on DMA, and the watchdog is kicked only when every task met its deadline |
| Telemetry | One 91-byte binary packet, specified in [`telemetry_packet_spec.md`](assets/firmware/telemetry_packet_spec.md) |
| Assembly | JLCPCB PCBA plus a deliberate hand-solder list |

## Docs

| File | What's in it |
|---|---|
| [`bom/`](assets/bom/) | Full BOM, the machine-assembled and hand-solder subsets, and why each part was picked |
| [`power/power_budget.csv`](assets/power/power_budget.csv) | Average and peak current per load |
| [`mechanical/mechanical_constraints.md`](assets/mechanical/mechanical_constraints.md) | How the 3U envelope sets the board size and stacking order |
| [`pcb/stackup_and_layout.md`](assets/pcb/stackup_and_layout.md) | Stackup, floorplan, grounding and RF routing rules |
| [`firmware/firmware_architecture.md`](assets/firmware/firmware_architecture.md) | Scheduler, task table, fault handling, the M7 state machine |
| [`testing/test_plan.md`](assets/testing/test_plan.md) | From the first power-on to the flight-day checklist |
