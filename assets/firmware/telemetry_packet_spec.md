# Telemetry packet specification v1.0

Binary, **little-endian**, CRC-16/CCITT-FALSE over every byte from `sync` up to but not including `crc16`.

Keep this file as the single source of truth. The flight firmware, the ground station parser and the synthetic packet generator all derive from it. When a field changes, change it **here first**.

## Packet types

| Type | Name | Rate | Purpose |
| --- | --- | --- | --- |
| 1 | TELEMETRY | 1 Hz | Everything below. The main stream. |
| 2 | EVENT | on occurrence | Launch detected, burst detected, landing detected, sensor fault, reset |
| 3 | MEMORY_REPORT | 1 per test cycle (~60 s) | Full M7 result: per-device error counts, addresses, bit positions, V_MEM sweep |
| 4 | STATUS | 0.1 Hz | Firmware version, uptime, config echo, SD free space |

## TELEMETRY packet layout (type 1)

| Offset | Type | Field | Units / encoding | Notes |
| --- | --- | --- | --- | --- |
| 0 | uint16 | sync | 0xA5C3 | Framing |
| 2 | uint8 | packet_type | 1 | |
| 3 | uint16 | seq | increments, wraps | Loss detection |
| 5 | uint32 | t_ms | ms since boot | Monotonic |
| 9 | uint32 | gps_utc | unix seconds | 0 = no fix |
| 13 | int32 | lat_1e7 | deg × 1e7 | |
| 17 | int32 | lon_1e7 | deg × 1e7 | |
| 21 | int32 | alt_gps_mm | mm above ellipsoid | **Primary altitude above 31 km** |
| 25 | uint8 | sats | count | M5 |
| 26 | uint8 | fix_type | 0/2/3 | M5 |
| 27 | uint16 | hdop_1e2 | HDOP × 100 | M5 |
| 29 | uint32 | pressure_pa | Pa | **Primary altitude below 31 km** |
| 33 | int16 | temp_ext_c100 | °C × 100 | PT1000, outside insulation — M1 |
| 35 | int16 | temp_int_c100 | °C × 100 | SHT45, internal air |
| 37 | int16 | temp_board_c100 | °C × 100 | TMP117 — M7 axis |
| 39 | uint16 | humidity_1e2 | %RH × 100 | Valid below ~10 km |
| 41 | int16[3] | accel_mg | mg | |
| 47 | int16[3] | gyro_dps10 | dps × 10 | |
| 53 | int16[3] | mag_ut10 | µT × 10 | Spin rate — M6 |
| 59 | uint16 | uva_uw_cm2 | µW/cm² | M4 |
| 61 | uint16 | uvb_uw_cm2 | µW/cm² | M4 — the headline channel |
| 63 | uint16 | uvc_uw_cm2 | µW/cm² | M4 |
| 65 | uint16 | pm1_ugm3_10 | µg/m³ × 10 | 0xFFFF = sensor off |
| 67 | uint16 | pm25_ugm3_10 | µg/m³ × 10 | M2 |
| 69 | uint16 | pm10_ugm3_10 | µg/m³ × 10 | M2 |
| 71 | uint16 | geiger_cpm | counts/min | **M3 — the headline result** |
| 73 | uint32 | geiger_total | cumulative counts | |
| 77 | uint16 | vbat_mv | mV | |
| 79 | uint16 | v3v3_mv | mV | INA226 |
| 81 | uint16 | vmem_mv | mV | INA226 — M7 |
| 83 | uint16 | i_main_ma_10 | mA × 10 | INA226 |
| 85 | uint16 | mem_err_count | cumulative | M7 |
| 87 | uint8 | reset_reason | bitfield from RCC_CSR | M7 forensics |
| 88 | uint8 | status_flags | see below | |
| 89 | uint16 | crc16 | CRC-16/CCITT-FALSE | |

**Total: 91 bytes.** At 1 Hz that is 91 B/s, comfortably inside the LR900-A's ~2.1 kB/s and the Delock's slower air rates. Headroom is deliberate: the link degrades badly at range and you want the packet to still fit at a lower spreading factor.

### status_flags bitfield

| Bit | Meaning |
| --- | --- |
| 0 | sd_ok |
| 1 | gps_ok (3D fix) |
| 2 | sps30_on |
| 3 | radio2_ok |
| 4 | geiger_ok |
| 5 | brownout_seen (sticky) |
| 6 | wdt_reset (sticky) |
| 7 | memory_test_running |

### reset_reason bitfield (from RCC_CSR)

| Bit | Source |
| --- | --- |
| 0 | Low-power reset |
| 1 | Window watchdog |
| 2 | Independent watchdog |
| 3 | Software reset |
| 4 | POR/PDR |
| 5 | NRST pin |
| 6 | Brown-out reset |

Read `RCC_CSR` **at boot, before clearing it**, copy into backup SRAM, then clear. Without this, M7 cannot distinguish a genuine memory fault from a reset.

## Design notes

**Why fixed-point and not float.** Halves the packet size, removes any endianness or NaN ambiguity, and makes the parser trivially testable. Scale factors are chosen so each field's resolution comfortably exceeds the sensor's own.

**Why a 16-bit CRC and not a checksum.** CRC-16/CCITT-FALSE catches all single- and double-bit errors and all burst errors up to 16 bits. On a noisy LoRa link near the sensitivity limit you will receive corrupted packets, and silently logging a corrupted altitude is worse than dropping the packet.

**Why `seq` wraps rather than saturating.** Loss statistics are computed modulo 65536 on the ground. A saturating counter loses information after 18 hours at 1 Hz.

**Why both radios send the same stream.** The ground station deduplicates on `(seq, packet_type)` and records which radio delivered each packet plus its RSSI. That comparison **is** mission M6 — you get the link-budget dataset for free from redundancy you wanted anyway.
