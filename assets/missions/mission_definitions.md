# Mission definitions

Six missions. Four have a public benefit you can state in one sentence to a non-specialist. Two are free with hardware needed anyway.

Every mission below is written as: hypothesis → measurement → the graph → what would falsify it → limitations.

---

## M1 — Upper-air profile over Uzbekistan

**Public benefit.** Central Asia has sparse upper-air observation coverage. Numerical weather prediction over the region relies heavily on model output rather than measurement. Every real profile is a data point that can be compared against the models people forecast from.

| | |
| --- | --- |
| **Hypothesis** | A student-built radiosonde-class payload can produce a temperature, pressure, humidity and wind profile whose deviation from ERA5 reanalysis is quantifiable and bounded. |
| **Independent variable** | Altitude (pressure below 31 km, GNSS above) |
| **Dependent variables** | External air temperature, humidity, horizontal wind speed and direction |
| **Sensors** | PT1000 + MAX31865, SHT45, MS5611, SAM-M10Q |
| **Sampling** | 1 Hz. At 5 m/s ascent that is one sample every 5 m of altitude. |
| **Accuracy needed** | ±0.5 °C temperature, ±1 hPa pressure, ±1 m/s wind |
| **The graph** | Measured temperature against altitude, overlaid with the ERA5 profile for the same time and place. Second panel: wind speed and direction against altitude. |
| **Supports the hypothesis** | The tropopause appears at the right altitude (~11 km over Uzbekistan in most seasons) with the right temperature minimum (~−55 to −65 °C), and the lapse rate matches ERA5 within a few °C. |
| **Contradicts it** | No tropopause visible, lapse rate wildly wrong, or wind bearing inconsistent with the balloon's actual drift. |
| **Alternative explanations to rule out** | Solar heating of the probe (shade it, and compare ascent against descent); probe thermal lag (check the response time against ascent rate); GNSS velocity error (cross-check against the ground track). |
| **Limitations** | One profile at one time and place. Not a climatology. The humidity channel is only meaningful below ~10 km. |

**Note on wind.** The balloon is a Lagrangian tracer — its horizontal GNSS velocity *is* the wind velocity. That is precisely how an operational radiosonde measures wind. You get this mission's second variable free from a module you needed for recovery.

---

## M2 — Boundary-layer air quality profile over Tashkent

**Public benefit.** Tashkent has severe seasonal particulate pollution. Ground stations measure concentration at one height. Almost nobody measures **how high the polluted layer extends** or where the temperature inversion caps it — which is what determines whether pollution disperses or accumulates.

| | |
| --- | --- |
| **Hypothesis** | Particulate concentration falls sharply at a specific altitude that coincides with a temperature inversion, and that altitude is measurable on both ascent and descent. |
| **Independent variable** | Altitude, 0–5 km |
| **Dependent variables** | PM1, PM2.5, PM10; air temperature |
| **Sensors** | SPS30, PT1000, MS5611 |
| **Sampling** | 1 Hz. At 5 m/s that is a sample every 5 m — ample to resolve an inversion typically 300–1500 m up. |
| **The graph** | PM2.5 against altitude on the x-axis with temperature on a second x-axis, both against altitude on y. The inversion and the concentration drop should appear at the same height. Ascent and descent plotted separately. |
| **Supports the hypothesis** | A clear concentration step, co-located with a temperature inversion, reproduced on both ascent and descent. |
| **Contradicts it** | Concentration falls smoothly with no step, or the step and the inversion are at clearly different altitudes. |
| **Alternative explanations** | Sensor response lag (compare ascent and descent — a lag shifts them in opposite directions); the payload passing through a local plume; humidity affecting optical scattering (log RH and check). |
| **Limitations — state these plainly** | **The SPS30 is valid only below roughly 5 km.** Sensirion publishes no low-pressure specification; the fan's volumetric flow and the mass-concentration algorithm both assume near-sea-level air density. Above ~5 km the readings are qualitative at best. The sensor is powered down above 6 km. |

This limitation is not a weakness. The boundary layer is exactly where the public-health signal lives.

---

## M3 — Cosmic radiation dose from ground to burst

**Public benefit.** Aircrew are classified as occupationally radiation-exposed under EU and ICRP frameworks, and dose at cruise altitude is regulated and monitored. This measures what crew and passengers on Central Asian routes actually fly through, and recovers the Regener–Pfotzer maximum along the way.

| | |
| --- | --- |
| **Hypothesis** | Secondary cosmic ray count rate increases by roughly an order of magnitude from ground level, peaks near 15–20 km, and declines above that. |
| **Independent variable** | Altitude |
| **Dependent variable** | Geiger count rate (CPM), and an approximate dose rate derived from it |
| **Sensors** | SBM-20 Geiger module, MS5611, SAM-M10Q |
| **Sampling** | 1 Hz count readout, analysed in 30–60 s bins for adequate counting statistics |
| **The graph** | **Count rate against altitude.** This is the single most visually convincing plot the payload will produce — a textbook curve with an unmistakable peak. Mark cruise altitude (10–12 km) on it. |
| **Supports the hypothesis** | Count rate rises roughly 10–20× from ground, peaks between 15 and 20 km, then declines. |
| **Contradicts it** | Flat profile, or a peak at clearly the wrong altitude. |
| **Alternative explanations** | Temperature affecting the tube's plateau voltage (log board temperature and check for correlation); HV supply sagging as the battery drains (log VBAT); counting statistics at low rates (use wide enough bins and show error bars). |
| **Limitations** | An SBM-20 is uncalibrated for dose. Converting CPM to µSv/h requires an assumed conversion factor and an assumed radiation spectrum, so quote the dose figure as **approximate and order-of-magnitude**, alongside the raw count rate which is the real measurement. A GM tube does not discriminate particle type or energy. |

**Say the honest thing in your write-up:** the count rate is a solid measurement; the dose conversion is an estimate. Presenting both is more credible than presenting only the dose.

---

## M4 — UV irradiance and ozone attenuation against altitude

**Public benefit.** Quantifies how much UV the ozone layer removes above a given altitude, and therefore what UV dose aircrew receive at cruise. Also gives an ozone-column attenuation proxy from a $9 sensor.

| | |
| --- | --- |
| **Hypothesis** | UVB irradiance rises far more steeply with altitude than UVA, because UVB is preferentially absorbed by ozone, and the UVB/UVA ratio therefore increases with altitude. |
| **Independent variable** | Altitude |
| **Dependent variables** | UVA, UVB, UVC irradiance (three separate channels) |
| **Sensors** | AS7331, MS5611, SAM-M10Q, IMU (for sun-angle correction) |
| **Sampling** | 1 Hz |
| **The graph** | Three irradiance curves against altitude on one panel; the **UVB/UVA ratio** against altitude on a second. The ratio panel is the actual result. |
| **Supports the hypothesis** | UVB rises steeply above ~15 km while UVA rises gently; the ratio increases monotonically with altitude. |
| **Contradicts it** | Both channels scale together (suggesting the ratio is driven by geometry, not absorption), or UVB shows no altitude dependence. |
| **Alternative explanations** | **Payload spin and tumbling changing the sensor's angle to the sun** — this is the big one. Use IMU and magnetometer data to reconstruct orientation and either correct for it or select only near-nadir-stable intervals. Also: cloud reflection below, and solar elevation changing during the flight. |
| **Limitations** | The sensor has a cosine response that is imperfect off-axis. Without a stabilised platform this is a relative measurement, not an absolute irradiance. **Requires a quartz or PTFE window** — ordinary plastic blocks UVB and UVC entirely and you would measure nothing without knowing why. |

---

## M5 — GNSS availability and accuracy against altitude

**Public benefit.** Navigation integrity for aviation and UAV operations. Free — no extra hardware.

| | |
| --- | --- |
| **Hypothesis** | Satellite count and horizontal dilution of precision improve with altitude as terrain masking disappears, with no degradation through the stratosphere. |
| **Measured** | Satellite count, fix type, HDOP, and the disagreement between GNSS altitude and pressure altitude |
| **The graph** | Satellite count and HDOP against altitude; a second panel showing GNSS-minus-barometric altitude against altitude, which shows exactly where the barometer leaves its specified range. |
| **Supports it** | Satellite count rises through the first few km then plateaus; HDOP improves and stays low. |
| **Contradicts it** | Fix degrades or is lost at altitude — which, if it happens above 18 km, almost certainly means the airborne dynamic model was not set. |
| **Limitations** | One receiver, one flight, one constellation configuration. |

---

## M6 — LoRa link budget against slant range and altitude

**Public benefit.** Emergency and remote-area communications design, and high-altitude platform links. Free — you already have two radios and a magnetometer.

| | |
| --- | --- |
| **Hypothesis** | Received signal strength follows free-space path loss within a few dB once the payload is above the horizon, and the residual variation is explained by antenna orientation. |
| **Measured** | RSSI and SNR per packet on both radios, packet loss rate, slant range computed from GNSS, payload orientation from IMU and magnetometer |
| **The graph** | RSSI against slant range with the theoretical free-space path loss curve overlaid; a second panel of RSSI residual against payload roll angle, which should show the dipole's nulls. |
| **Supports it** | Measured RSSI tracks the FSPL slope; residuals correlate with orientation in the pattern a dipole predicts. |
| **Contradicts it** | RSSI far below prediction (suggesting a matching or antenna problem), or residuals uncorrelated with orientation. |
| **Limitations** | Ground station antenna pointing is manual and imperfect. Two different radios in two different bands are not a controlled comparison — note this rather than over-claiming. |

---

## M7 — COTS memory and electronics reliability under flight stress

**Public benefit.** Cheap commercial parts are increasingly flown in UAVs, high-altitude platforms and small satellites. Establishing how their retention margin behaves across the real thermal range is directly useful to anyone doing that.

| | |
| --- | --- |
| **Hypothesis** | The minimum supply voltage at which an SRAM retains data varies measurably and monotonically with temperature, and that relationship holds across the flight's thermal range. |
| **Independent variables** | Board temperature (primary), altitude, V_MEM |
| **Dependent variable** | Retention voltage threshold per device; error count, address, bit position |
| **Sensors** | TMP117, 2× NTC, 2× INA226, MS5611, SAM-M10Q |
| **The graph** | Retention voltage threshold against board temperature, one curve per memory technology (SRAM, FRAM, NOR, MRAM). |
| **Supports it** | A clean, repeatable, monotonic curve; SRAM shows a clear threshold while FRAM and MRAM do not (as expected — different physics). |
| **Contradicts it** | Threshold scattered or non-monotonic, implying rail noise or a measurement artefact rather than a device property. |
| **Alternative explanations to rule out — this is the heart of the experiment** | Rail dip during the test (INA226 logs V_MEM and 3V3 min continuously); a reset mid-test (RCC_CSR flags into backup SRAM at every boot); radio TX transient (correlate error timestamps against TX timestamps); a firmware bug (the pattern is regenerated from a seed, never stored, so a corrupted stored copy cannot masquerade as a device error); SPI signal integrity (the same test at nominal 3.3 V must give zero errors — if it does not, it is a hardware fault, not science). |

### The cosmic-ray upset claim — set expectations now

Bulk CMOS SRAM soft-error rate at sea level is roughly 100–1000 FIT/Mbit. Atmospheric neutron flux peaks near 15–20 km at roughly 300–500× sea level. With ~8 Mbit under test and ~2 hours above 15 km:

```
200 FIT/Mbit x 8 Mbit x 500 x 2 h  =  1.6e-3 expected upsets
```

**That is about one chance in six hundred of seeing a single event.** Expect zero.

Report it as an **upper bound**: *"Zero upsets observed in N Mbit-hours above 15 km, giving a rate below X, consistent with the expected ~10⁻³."* That is an honest, defensible statement that a reviewer will accept.

Claiming a single observed anomaly was caused by radiation — when a rail dip, EMI, temperature or a firmware bug are each orders of magnitude more likely — is the mistake that gets a student result dismissed. The instrumentation above exists specifically so you can rule those out, and so that your null result is a *measurement* rather than an absence.

---

## Mission trade-offs and cut order

| Mission | Extra hardware | Cost | Guaranteed result? | Cut priority |
| --- | --- | --- | --- | --- |
| M1 Weather profile | none beyond core | $0 | Yes | Never — core |
| M5 GNSS integrity | none | $0 | Yes | Never — free |
| M6 Link budget | none | $0 | Yes | Never — free |
| M7 Memory reliability | memory ICs + LDO + INA226 | ~$45 | Yes (the retention curve) | Keep — it is the engineering depth |
| M3 Radiation dose | Geiger module | $28 | **Yes — the strongest single graph** | Keep |
| M4 UV / ozone | AS7331 + quartz window | $21 | Likely, if orientation is handled | Cut second |
| M2 Air quality | SPS30 | $47 | Yes, below 5 km | **Cut first** — largest, heaviest, most power |

If the schedule compresses, cut M2 then M4. M1, M3, M5, M6 and M7 use parts that are already CRITICAL for other reasons, so they cost you nothing to keep.
