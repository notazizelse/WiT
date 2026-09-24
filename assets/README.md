# WIT — 3U stratospheric balloon payload · assets

Reference material for the electronics, PCB, firmware and mission design.
Generated 2026-09-22. Nothing here touches the KiCad schematic or board files.

## Contents

```
assets/
├── README.md                       this file
├── bom/
│   ├── BOM_full.csv                every part, 22 columns, 74 lines
│   ├── BOM_pcb_jlcpcb.csv          machine-assembled subset (56 lines, $89/board)
│   ├── BOM_handsolder.csv          you fit these yourself (15 lines, $125)
│   ├── BOM_offboard_from_full.csv  off-board items extracted from the full list
│   ├── BOM_external_system.csv     battery, antennas, structure, ground station, recovery
│   ├── BOM_tools_test_equipment.csv  what you own, what to buy, in priority order
│   └── BOM_notes.md                WHY each part, and why not the alternatives
├── power/
│   └── power_budget.csv            per-load average and peak current, with margins
├── mechanical/
│   └── mechanical_constraints.md   100×100×300 envelope → board size, height and mass budgets
├── pcb/
│   └── stackup_and_layout.md       4-layer stackup, floorplan, grounding, RF routing, DRC, fab outputs
├── learning/
│   └── pcb_learning_path.md        2-layer -> 4-layer: KiCad vs Altium, ~9h of verified videos, practice plan
├── firmware/
│   ├── firmware_architecture.md    scheduler, tasks, memory experiment state machine, fault handling
│   └── telemetry_packet_spec.md    91-byte binary packet, the single source of truth
├── testing/
│   └── test_plan.md                Stage 0 (before power) through flight-day checklist
├── missions/
│   └── mission_definitions.md      7 missions: hypothesis, graph, falsification, limitations
├── procurement/
│   └── suppliers.md                local Tashkent / distributors / AliExpress / JLCPCB, and order sequence
├── datasheets/                     (empty — fill as you download)
├── reference_designs/              (empty)
└── app_notes/                      (empty)
```

## Confirmed decisions

| Item | Decision |
| --- | --- |
| Envelope | 3U, 100 × 100 × 300 mm |
| Board | 4-layer, JLC04161H-7628 stackup, ENIG, ~90 × 90 mm |
| MCU | STM32F407VET6, LQFP-100 |
| Radios | **Both** — MicoAir LR900-A (5 V, external) and **Ebyte E22-900T22S** (SMD, 850–930 MHz), separate UARTs |
| Battery | 4 × Energizer L91 AA — **−40 °C rating confirmed from packaging, available locally** |
| Geiger | CAJOE module + SBM-20 — **confirmed orderable from AliExpress** |
| Camera | None on this PCB |
| Assembly | JLCPCB PCBA + a deliberate hand-solder list |
| EDA | **KiCad 9** — stay here for this board; learn Altium after it flies |
| Power | Single buck-boost → 5 V, then LC + LDO → 3.3 V (one hot loop, quiet rail for M7) |

## Costs

| | USD |
| --- | --- |
| PCB components, per board | ~168 |
| Bare PCB + assembly, per board at qty 5 | ~43 |
| **Assembled board, each** | **~211** |
| 5 assembled boards | 1055 |
| External flight hardware, ground station, recovery | ~600 |
| Test equipment still to buy (priority items) | ~245 |
| Spares, consumables, shipping and customs | ~400 |
| **Project total** | **~2300** |

Component list is at **v2** — see `bom/COMPONENT_CHANGES_v2.md` for the eight availability changes.

## Open items blocking progress

1. **Does 100 × 100 mm mean the structural box or the finished insulated exterior?** This sets the board width at either 90 mm or 70 mm. See `mechanical/mechanical_constraints.md` §1.
2. Structure material, wall thickness, tube or box.
3. Launch date.
4. Whether a competition imposes requirements.
5. Frequency council decision (in progress).
6. Balloon payload mass/aviation clearance — same authority, separate question.

## Order these today

Nothing below depends on the PCB being finished.

1. **STM32F407VET6 dev board × 2** (~$24, AliExpress) — the largest schedule lever in the project
2. **Geiger module + SBM-20** (AliExpress) — longest, least predictable lead time
3. **u-blox SAM-M10Q × 2** (Mouser/DigiKey) — longest distributor risk
4. **MicoAir LR900-A × 2** — stock already uncertain
5. **Bench PSU + ST-LINK × 2 + multimeter + logic analyser** — needed *before* the board arrives

See `procurement/suppliers.md` for the full sequence and supplier links.

## The three things most likely to ruin the flight

1. **Not setting the GNSS airborne dynamic model at every boot.** Every measurement above 18 km is lost, silently. `firmware/firmware_architecture.md`.
2. **Using a BMP-series barometer.** Specified only to 300 hPa ≈ 9 km. `bom/BOM_notes.md`.
3. **Transmitting on both radios simultaneously.** Desense or front-end damage. Needs a firmware interlock, not a convention.
