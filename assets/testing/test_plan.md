# Test plan — first power-up to flight qualification

## Stage 0 — Before power (30 minutes, no exceptions)

The board has arrived. Do not connect anything yet.

| # | Check | How | Fail action |
| --- | --- | --- | --- |
| 1 | Visual inspection under magnification | Every fine-pitch part: LQFP-100, LGA-14 IMU, QFN-8 baro, all QFN regulators | Rework with hot air |
| 2 | Solder bridges | Sweep every fine-pitch edge with the loupe or USB microscope | Wick and reflow |
| 3 | Component orientation | Every polarised part against the assembly drawing: ICs, diodes, LEDs, tantalums | **Stop.** A backwards IC will destroy itself on power-up |
| 4 | Missing / tombstoned parts | Compare against the placement drawing | Hand-fit from spares |
| 5 | **VBAT to GND resistance** | Multimeter, both polarities | Below ~100 Ω means a short. Find it before applying power. |
| 6 | **3V3 to GND resistance** | Multimeter | Expect a few kΩ rising as caps charge. A dead short is a fault. |
| 7 | **5V0 to GND resistance** | Multimeter | Same |
| 8 | **V_MEM to GND resistance** | Multimeter | Same |
| 9 | Connector orientation and keying | Physically test-fit every mating cable | Correct before flight wiring exists |

## Stage 1 — First power-up (current-limited)

**Use the bench supply. Not a battery. This is what the bench supply is for.**

```
1. Set 6.0 V, current limit 100 mA. Output OFF.
2. Connect to J1. Nothing else connected. No SD card. No modules.
3. Output ON. Watch the current meter, not the board.

   Current runs to the 100 mA limit  -> OFF IMMEDIATELY. Short. Stage 0 missed it.
   Current 10-40 mA and stable       -> correct. Continue.
   Current < 5 mA                    -> regulator not starting. Check enable pins.
```

| # | Measure | Expected | Notes |
| --- | --- | --- | --- |
| 1 | 3V3 test point | 3.30 V ±3 % | If wrong, check the feedback divider before anything else |
| 2 | 5V0 test point | 5.00 V ±3 % | |
| 3 | V_MEM test point | ~3.3 V at DAC reset value | |
| 4 | VDDA after the ferrite | = 3V3 | |
| 5 | Board temperature by hand after 2 min | Nothing hot | A hot IC is a fault even if voltages look right |
| 6 | Raise the current limit to 500 mA, repeat | Same voltages | |
| 7 | Sweep input 4.4 → 7.2 V | All rails hold | This is the real battery range |

**Then, and only then:** 3V3 ripple on the scope (expect < 50 mV pk-pk), and the same on V_MEM (want < 10 mV — this rail carries mission M7).

## Stage 2 — MCU bring-up

| # | Step | Success criterion |
| --- | --- | --- |
| 1 | Connect ST-LINK, attempt SWD connect under reset | Target detected, correct device ID |
| 2 | Read the device ID and flash size registers | Matches STM32F407VE |
| 3 | Flash a blink on an LED | LED blinks |
| 4 | Check the blink period against a stopwatch over 60 s | Within 1 % → HSE is running, PLL configured correctly |
| 5 | UART console "hello" at 115200 | Readable on the terminal |
| 6 | Read and print RCC_CSR reset flags | Sensible values |
| 7 | Enable IWDG, deliberately stop kicking it | Board resets; RCC_CSR shows IWDG reset |
| 8 | Verify the LSE and RTC tick | RTC seconds advance |
| 9 | Verify backup SRAM survives a reset | Write a pattern, reset, read it back |

If step 4 fails, the HSE crystal or its load capacitors are wrong. Check C0G, not X7R.

## Stage 3 — Peripheral bring-up, ONE AT A TIME

**Never power up everything at once on a first board.** Bring up one peripheral, confirm it, move on. If something draws unexpected current you want exactly one candidate.

| Order | Subsystem | Test | Expected |
| --- | --- | --- | --- |
| 1 | I²C bus scan | Scan both buses | Every expected address responds, no unexpected ones |
| 2 | TMP117 | Read temperature | Within 2 °C of room temperature |
| 3 | SHT45 | Read T and RH | Plausible; breathe on it and watch RH rise |
| 4 | MS5611 | Read pressure, convert to altitude | Within ~50 m of Tashkent's ~450 m elevation |
| 5 | ICM-42688-P | Read WHO_AM_I, then accel | 0x47; accel reads ~1 g on one axis, rotate and confirm it moves |
| 6 | MMC5983MA | Read magnetometer | Field magnitude ~25–65 µT; rotating changes the axes |
| 7 | AS7331 | Read UV channels | Near zero indoors; take it outside and UVA rises sharply |
| 8 | MAX31865 + PT1000 | Read temperature | Matches TMP117 within 1 °C; warm the probe and watch it track |
| 9 | INA226 ×2 | Read bus voltage and current | Matches the bench supply readout |
| 10 | V_MEM sweep | DAC ramp, read back on INA226 | V_MEM follows the DAC across 0.8–3.3 V, monotonic |
| 11 | SPI memory devices | Read JEDEC ID from each | Each device returns its correct manufacturer/device ID |
| 12 | SPI memory write/read | Pattern write and verify at 3.3 V | Zero errors. **Any error here is a hardware fault, not science.** |
| 13 | microSD | Mount, write a file, read it back | File intact |
| 14 | GNSS | Outdoors, cold start | 3D fix within 60 s; position within 10 m of known |
| 15 | **GNSS airborne mode** | Set DYNMODEL=8, poll it back | **Poll returns 8. Verify this, do not assume it.** |
| 16 | GNSS 1PPS | Scope or timer capture | 1.000 s period, clean edge |
| 17 | Geiger module | Connect, count for 10 min | Background 15–30 CPM for an SBM-20. Bring a check source if you can. |
| 18 | SPS30 | Read PM values | Near zero in clean air; rises sharply near a candle or incense |
| 19 | Delock 12592 | Loopback, then link to the ground unit across the room | Packets received, CRC valid |
| 20 | MicoAir LR900-A | Same | Packets received |
| 21 | **Measure LR900-A TX current** | Scope across a shunt, or the bench supply meter | **This resolves the UNKNOWN in the power budget. Record it.** |
| 22 | Both radios, interlock test | Attempt simultaneous TX in firmware | The interlock must prevent it. Verify in the logic analyser. |

## Stage 4 — Integrated bench test

| # | Test | Duration | Pass criterion |
| --- | --- | --- | --- |
| 1 | All subsystems running together | 4 h | No resets, no watchdog events, no I²C hangs |
| 2 | Measure actual total current | — | Within 20 % of the power budget. Update the budget with real numbers. |
| 3 | Full-rate logging to SD | 4 h | No dropped records, file intact, matches the transmitted stream |
| 4 | Telemetry to the ground station | 4 h | Packet loss < 1 % at 10 m |
| 5 | Memory test cycles | 4 h | Retention threshold repeatable within 50 mV at constant temperature |
| 6 | **Deliberate brownout** | — | Drop the supply to 2.5 V for 100 ms. Board must reset cleanly, log the brownout, resume, and preserve backup SRAM contents. |
| 7 | **Deliberate SD removal mid-write** | — | Board must not hang; `sd_ok` clears; logging continues to flash |
| 8 | Battery run-down | to cutoff | Confirms real endurance against the predicted 3× margin |

## Stage 5 — Environmental

| Test | Method | Pass criterion |
| --- | --- | --- |
| **Cold soak −20 °C** | Domestic freezer, 4 h, board running and logging | All sensors read; no resets; retention threshold shifts smoothly, not abruptly |
| **Cold soak −60 °C** | Dry ice in an insulated box, 2 h | Same. This is the qualification test. |
| **Thermal cycle** | Room → −20 °C → room, 5 cycles | No intermittent faults; no cracked joints; check the crystals still start cold |
| **Cold start** | Power on while already at −60 °C | Board boots. Crystals starting at temperature is a real failure mode. |
| **Low pressure** | Vacuum chamber to ~50 mbar, 1 h | MS5611 tracks; no arcing near the Geiger module; no component outgassing |
| **Vibration** | Mount on a vibrating surface or drive on a rough road, 30 min | No intermittent resets; SD card stays seated; check every connector after |
| **Drop / shock** | 1 m drop onto grass in the flight enclosure | Survives. This simulates landing. |
| **Insulation validation** | Full payload in the freezer, log internal vs external T | Confirms (or refutes) the 30–50 °C rise prediction. **This is a result, not just a test.** |
| **RF range** | Two hilltops or across a valley, line of sight, ≥5 km | Link closes with margin. Record RSSI vs distance — it is M6 baseline data. |

## Stage 6 — Near-flight rehearsal

Full payload, flight configuration, outdoors, on the actual launch day procedure.

1. Assemble exactly as for flight, including insulation, parachute and tracker.
2. Power on, run the full pre-launch checklist.
3. Leave it running on the ground for the full expected flight duration plus 2 h.
4. Ground station operating, chase-car setup, dashboard on the hotspot.
5. Walk or drive the payload several km away, tracking it live the whole time.
6. Recover it as if it had landed — find it using the tracker, no cheating.
7. Extract and analyse the data exactly as you will after the real flight.

**Anything that goes wrong here goes wrong on flight day too, but cheaper.**

## Stage 7 — Flight day checklist

**T−24 h**
- Fresh L91 cells fitted and voltage checked (each cell ≥ 1.7 V open circuit)
- SD card formatted and a test log written and verified
- Firmware version recorded; git hash confirmed in the STATUS packet
- Ground station tested end to end
- Weather and predicted landing site checked

**T−2 h**
- Payload assembled, all connectors seated and secured
- Conformal coating masks removed from MS5611 port, SPS30 inlet, UV window
- Antennas fitted and continuity checked
- Independent tracker powered on and confirmed reporting

**T−30 min**
- Power on, confirm GNSS 3D fix
- **Confirm the airborne dynamic model is set** (poll it, read the STATUS packet)
- Confirm telemetry on both radios
- Confirm SD logging
- Confirm Geiger background count is plausible
- Photograph the assembled payload

**T−5 min**
- Final telemetry check
- Seal the enclosure
- Confirm the parachute and train are correctly attached

**Post-recovery**
- Photograph before opening
- Power down deliberately; do not pull the battery
- Copy the SD card **twice, to two different machines, before anything else**
- Then dump the W25Q128 backup log
- Only then start analysis
