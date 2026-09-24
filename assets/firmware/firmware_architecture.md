# Firmware architecture — STM32F407VET6

## Bare metal with a cooperative scheduler, not FreeRTOS

FreeRTOS is the reflex answer and it is the wrong one here.

| | Bare metal + timer scheduler | FreeRTOS |
| --- | --- | --- |
| Determinism | Fully predictable | Depends on priorities and blocking |
| Debugging a fault at 25 km | One call stack | N stacks, priority inversion, stack overflow |
| Memory overhead | ~0 | ~5–10 KB plus per-task stacks |
| Failure modes you must reason about | Few | Deadlock, starvation, stack overflow, priority inversion |
| Development time | Lower for this workload | Higher |

Your workload is a handful of periodic tasks at 1–100 Hz with no blocking I/O that cannot be made non-blocking with DMA. That is exactly what a cooperative scheduler handles well. **Reliability over feature count** — RTOS bugs are hard to reproduce, and you cannot attach a debugger at 25 km.

## Structure

```
main()
 ├── hw_init()            clocks, GPIO, peripherals, IWDG
 ├── boot_forensics()     read RCC_CSR -> backup SRAM, increment boot counter
 ├── config_gnss()        UBX: airborne <1g dynamic model  <-- CRITICAL
 ├── self_test()          probe every sensor, set status_flags
 └── while(1) scheduler_run()

scheduler: 1 kHz SysTick tick, table of {period_ms, last_run, fn}
```

### Task table

| Task | Rate | Budget | Job |
| --- | --- | --- | --- |
| `task_imu` | 100 Hz | 1 ms | Read ICM-42688 FIFO over SPI+DMA. Ring buffer for vibration FFT. |
| `task_baro` | 10 Hz | 2 ms | MS5611 conversion state machine (it needs ~9 ms per conversion — never block). |
| `task_env` | 1 Hz | 3 ms | SHT45, TMP117, NTCs, MAX31865, AS7331. |
| `task_gnss` | 1 Hz | 2 ms | Parse UBX from UART DMA ring buffer. Verify dynamic model still set. |
| `task_pm` | 1 Hz | 2 ms | SPS30 read. Power down above 6 km, power up below. |
| `task_geiger` | 1 Hz | <1 ms | Read the timer counter, compute CPM over a sliding window. |
| `task_power` | 1 Hz | 2 ms | Both INA226s. Track rail min/max between samples. |
| `task_memtest` | continuous, sliced | 5 ms/slice | The M7 state machine. Never blocks. Details below. |
| `task_log` | 5 Hz | 10 ms | Write to SD (SDIO+DMA) and mirror to W25Q128. |
| `task_telem` | 1 Hz | 5 ms | Build packet, CRC, push to both radio UARTs via DMA. |
| `task_health` | 1 Hz | 1 ms | Kick IWDG, evaluate flags, detect launch/burst/landing. |

**Total worst case ≈ 35 ms per second of an 84 MHz budget.** Enormous headroom, deliberately.

### Rules that keep it deterministic

1. **No task blocks. Ever.** Every driver is a state machine driven by the scheduler. `HAL_Delay()` does not appear anywhere after `hw_init()`.
2. **All bus traffic is DMA.** SPI, SDIO and both radio UARTs.
3. **Two I²C buses.** Bus 1: environmental sensors. Bus 2: power monitors. A single hung device cannot take down the whole sensor set. Implement an I²C bus recovery routine (clock out 9 bits, generate STOP) and call it on timeout.
4. **Watchdog kicked from one place only** — `task_health`, and only if every other task ran within its deadline. A watchdog kicked unconditionally from the main loop protects nothing.

## The memory experiment state machine (M7)

Sliced across scheduler calls so it never blocks. One full cycle ≈ 60 s.

```
IDLE -> WRITE_PATTERN -> SETTLE -> TRISTATE -> STEP_DOWN -> HOLD
     -> RESTORE -> READ_BACK -> CLASSIFY -> REPORT -> IDLE
```

| State | What happens |
| --- | --- |
| WRITE_PATTERN | V_MEM = 3.3 V. Write to all devices: alternating 0x55/0xAA, walking ones, and a seeded PRNG block (so the expected data is regenerable, not stored). |
| SETTLE | 10 ms. |
| TRISTATE | All SPI pins to analog mode (Hi-Z). CS lines deasserted first. |
| STEP_DOWN | DAC ramps V_MEM down to the next test point. Sweep 3.3 → 0.8 V in 50 mV steps across successive cycles. |
| HOLD | Dwell 1 s. INA226 logs actual V_MEM and leakage current throughout. |
| RESTORE | V_MEM back to 3.3 V, settle 10 ms. |
| READ_BACK | Re-enable SPI, read every device, compare against the regenerated pattern. |
| CLASSIFY | For each mismatch record: device, address, bit position, direction (0→1 or 1→0). |
| REPORT | Emit a MEMORY_REPORT packet with V_MEM, temperature, pressure, altitude, 3V3 min since last cycle, and the error list. |

**In parallel, every cycle:** CRC32 the STM32's internal SRAM and Flash using the hardware CRC unit and compare against the value stored at boot. This is the internal-versus-external comparison, and it costs almost nothing.

**Three things that make the result defensible:**

- The pattern is **regenerated from a seed**, not stored. A stored copy could itself be corrupted, and you would never know which side flipped.
- **Rail voltage is logged with every result.** A bit error with a coincident rail dip is a power event, not a memory event, and you can prove it.
- **Reset reason is logged.** An error after an unlogged reset is meaningless.

## Fault handling

| Fault | Response |
| --- | --- |
| Sensor fails self-test | Clear its status bit, keep flying, keep retrying every 10 s. **Never halt.** |
| I²C bus hangs | Bus recovery routine, then re-init. Log the event. |
| SD write fails | Set `sd_ok = 0`, fall back to W25Q128 only, keep transmitting. |
| Both radios fail | Keep logging. The flight data is on the card and the flash. |
| HardFault / MemManage / BusFault | Handler writes fault registers (CFSR, HFSR, stacked PC and LR) to backup SRAM, then triggers a software reset. On the next boot, transmit that record. **A crash you can diagnose is worth ten you cannot.** |
| Watchdog reset | Detected at boot from RCC_CSR, logged, counter incremented in backup SRAM. |
| Brownout | Same. Sticky bit set in `status_flags` for the rest of the flight. |

## Flight phase detection

Drives SPS30 power, log rate and telemetry rate.

| Phase | Entry condition | Behaviour |
| --- | --- | --- |
| PRELAUNCH | default at boot | 1 Hz telemetry, SPS30 on, LEDs on |
| ASCENT | altitude rising > 1 m/s for 30 s | SPS30 on below 6 km, off above |
| FLOAT/BURST | vertical speed reverses, or accel spike > 5 g | Log at max rate for 60 s either side — the burst transient is a one-off and it is good data |
| DESCENT | falling > 2 m/s for 30 s | SPS30 back on below 6 km (second profile) |
| LANDED | vertical speed < 0.5 m/s and accel stable for 60 s | Drop to 0.1 Hz beacon, sleep everything else, maximise beacon life |

**LANDED mode matters more than it sounds.** The recovery phase can last hours and it is when you most need the radio alive.

## Build and tooling

| | Choice | Why |
| --- | --- | --- |
| Toolchain | `arm-none-eabi-gcc` + CMake | Reproducible, CI-able, no IDE lock-in |
| HAL | STM32 LL for hot paths, HAL for setup | LL is thin and predictable; HAL is fine for init |
| Pin config | STM32CubeMX for pin assignment only | Catches peripheral conflicts **before** you draw the schematic |
| Debug | ST-LINK V3 + `gdb` / `openocd` | |
| Version | Compile git hash into the STATUS packet | You will fly several builds. Know which one flew. |

## Development order (start today on the dev board)

1. Blink, clocks, SysTick, UART console, IWDG.
2. Scheduler skeleton with dummy tasks and deadline monitoring.
3. One driver at a time against a breakout board: MS5611 → TMP117 → SHT45 → ICM-42688 → AS7331 → INA226.
4. GNSS: UBX parser plus the airborne dynamic-model configuration. **Test this one hard** — it is the single highest-consequence line of firmware in the project.
5. SD logging with SDIO+DMA and a raw circular log (no FAT).
6. Telemetry packet builder, CRC, and both radio drivers.
7. Memory test state machine (needs the real PCB's DAC/LDO — stub V_MEM with a fixed rail on the dev board until then).
8. Flight phase detection, fault handlers, backup SRAM forensics.

Steps 1–6 are all doable on a €12 dev board with breakout sensors, in parallel with PCB layout and fabrication. That is the whole point of buying it today.
